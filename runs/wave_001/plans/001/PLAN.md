# Pocket Aviary v1 implementation plan

## 1. Product boundary and v1 scope

Pocket Aviary v1 is a web-only, single-user, single-aviary product centered on a small living scene with two starter birds and eventual growth to at most seven birds. The implementation should protect three non-negotiable experiences:

- the aviary appears to have been continuing before the user arrived;
- birds respond to honest presence and small gestures without becoming chores, counters, or pets to maintain;
- all state that matters to bird identity, mood, and long-term drift is canonical on the server.

V1 includes:

- email magic-link accounts with one canonical aviary per account;
- two starter birds at onboarding, stable bird identity, user-assigned names, and future age-gated bird additions up to seven;
- hidden server-owned personality vectors, mood persistence, server-side simulation ticks, and append-only interaction events;
- the main horizontal browser aviary scene with three perch zones, local day/night cycle, rare ambient weather, idle micro-motion, procedural calls, listen-in, offer, settle, and field notebook;
- multi-device sync via server snapshots;
- account export, account deletion, session revocation, email change verification, and privacy policy/account settings;
- optional read-only visit invitations, off by default and revocable;
- screen-reader narration, call captions, keyboard navigation, WCAG AA text contrast, and a designed reduced-motion rendering mode;
- operational telemetry and performance observability that exclude per-bird and per-account relationship data.

V1 explicitly excludes native apps, passwords, SSO, payments, multiple aviaries, shared aviaries, household accounts, user-customizable scenes, direct bird placement, public profiles, follows, comments, public discovery, leaderboards, achievements, levels, scores, streaks, push notifications, hunger, distress, death, visible personality stats, and any surface that turns presence into a metric for the user to manage.

Ambiguity calls:

- Build the visual scene with a browser-native 2D renderer, not a full game engine. The scene needs careful composition, animation, and audio timing, but it does not need physics, collisions, or large-world camera systems.
- Store simulation state in a relational database with JSON columns only for shape-flexible state snapshots and generated motif/narration payloads. Identity, account, event, and authorization records should remain normalized.
- Use server-generated state snapshots plus client interpolation. Do not attempt browser-to-browser synchronization.

## 2. System architecture

Use four deployable server concerns behind one product domain:

1. Web app and asset delivery:
   - Server-render the shell and critical bootstrap data.
   - Serve the initial app bundle and compact scene assets through CDN.
   - Inline or edge-cache the first state snapshot where possible to meet time-to-first-bird.

2. API service:
   - Handles auth, account settings, session management, aviary state reads, interaction event writes, notebook reads, account export requests, visit invitation flows, and accessibility/settings preferences.
   - Enforces account UUID usage in all non-auth boundaries.
   - Never accepts client-written personality or mood absolute values.

3. Simulation worker:
   - Runs a slow tick, approximately once per minute per active/non-deleted aviary.
   - Consumes append-only interaction events in order.
   - Computes mood transitions, drift deltas, call scheduling hints, perch choices, weather state, notebook candidates, and canonical snapshot updates.
   - Is the only writer of personality vectors and canonical bird state.

4. Email/background job worker:
   - Sends magic links, account export links, invite links, email-change verification links, and optional visit notifications for users who explicitly enable them.
   - Handles account hard-deletion after the 30-day soft-delete window.

Recommended stack shape:

- TypeScript for client and server shared schema types.
- PostgreSQL for accounts, birds, events, snapshots, visits, settings, and notebook entries.
- Redis or a managed queue for simulation tick scheduling, email jobs, export jobs, and rate limiting.
- Object storage for generated account export JSON files with short-lived signed URLs.
- WebAudio on the client for procedural calls.
- Canvas or WebGL-backed 2D scene rendering with a DOM top bar and accessible parallel surfaces. If using WebGL, keep a Canvas2D fallback only if it stays within the bundle budget; otherwise the supported-browser boundary should carry the constraint.

Architecture invariant: the client may render, interpolate, synthesize audio, collect presence signals, and submit interaction events. It may not decide personality drift, persist canonical mood, overwrite bird state, or infer state from local history after reconnect.

## 3. Core data model

Use synthetic UUIDs for every internal account reference. Email appears only on the account table and email delivery payloads.

Primary tables:

- `accounts`
  - `id uuid primary key`
  - `email_encrypted text not null`
  - `email_verified_at timestamptz not null`
  - `pending_email_encrypted text null`
  - `created_at`, `updated_at`
  - `soft_deleted_at null`
  - `hard_delete_after null`
  - `privacy_policy_version_acknowledged null`

- `sessions`
  - `id uuid primary key`
  - `account_id uuid`
  - `device_label text`
  - `token_hash text`
  - `created_at`, `last_seen_at`, `revoked_at`, `expires_at`
  - Per-device tokens are revocable. Tokens are never logged in raw form.

- `magic_links`
  - `id uuid`
  - `email_hash text`
  - `token_hash text`
  - `created_at`, `expires_at`, `consumed_at`
  - Expire after 15 minutes and invalidate immediately on use.

- `aviaries`
  - `id uuid primary key`
  - `account_id uuid unique`
  - `created_at`
  - `local_timezone text`
  - `bird_count_cap int default 7`
  - `canonical_revision bigint`
  - `last_tick_at timestamptz`
  - `current_snapshot jsonb`
  - `settled_until_reengaged bool`
  - `active_weather jsonb null`

- `birds`
  - `id uuid primary key`
  - `aviary_id uuid`
  - `species_key text`
  - `name text`
  - `adopted_at timestamptz`
  - `personality jsonb` with normalized hidden traits: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`
  - `mood text`
  - `mood_since timestamptz`
  - `perch_zone text`
  - `call_signature_seed text`
  - `visual_seed text`
  - `drift_version int`
  - `deleted_at null`

- `interaction_events`
  - `id uuid primary key`
  - `aviary_id uuid`
  - `account_id uuid`
  - `bird_id uuid null`
  - `client_event_id text`
  - `event_type enum`: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer_seed`, `offer_song_fragment`, `offer_still_pool`, `settle_start`, `settle_undo`, `settle_complete`, `visibility_visible`, `visibility_hidden`
  - `occurred_at_client timestamptz`
  - `received_at timestamptz`
  - `payload jsonb`
  - `consumed_by_tick_at null`
  - Unique `(account_id, client_event_id)` for idempotency.

- `presence_windows`
  - Derived by server from presence pings, not trusted as client totals.
  - `account_id`, `aviary_id`, `started_at`, `ended_at`, `source_event_ids jsonb`, `duration_seconds`
  - Presence is counted only when visible, focused, and recently active according to client signals and server sanity caps.

- `notebook_entries`
  - `id uuid`
  - `aviary_id uuid`
  - `created_at`
  - `entry_date date`
  - `body text`
  - `source_kind text`
  - `source_refs jsonb`
  - Sparse and read-only.

- `account_settings`
  - `account_id uuid`
  - `audio_enabled bool`
  - `call_captions_enabled bool`
  - `reduced_motion_override enum null`
  - `visit_notifications_enabled bool default false`
  - `local_timezone text`
  - `accessibility_narration_enabled bool`

- `visit_invites`
  - `id uuid`
  - `host_account_id uuid`
  - `aviary_id uuid`
  - `visitor_email_encrypted text`
  - `visitor_email_hash text`
  - `token_hash text`
  - `created_at`, `expires_at`, `used_at`, `revoked_at`
  - `last_visit_at null`

- `visit_sessions`
  - `id uuid`
  - `invite_id uuid`
  - `started_at`, `ended_at`
  - `last_snapshot_at`
  - `approx_duration_seconds`
  - Visitor sessions authorize read-only snapshots only and never write interaction events.

- `account_exports`
  - `id uuid`
  - `account_id uuid`
  - `requested_at`, `completed_at`, `expires_at`
  - `object_key text`
  - `status text`

Do not create tables for scores, achievements, streaks, public profiles, feed ranking, leaderboards, visit counts for public comparison, hunger, happiness meters, or visible trait summaries. Operational metrics should live in telemetry infrastructure, not in product tables.

## 4. API surface

All APIs are HTTPS JSON, versioned under `/api/v1`. Use response schemas shared with the TypeScript client. Error responses use matter-of-fact language and machine-readable error codes.

Auth and account:

- `POST /auth/magic-link`
  - Input: email.
  - Behavior: rate limit per email hash and IP, create 15-minute magic link, email it.
  - Response: generic success, never reveals account existence.

- `POST /auth/magic-link/consume`
  - Input: token.
  - Behavior: atomically validate unexpired unused token, create account if needed, create starter aviary and two starter birds for new accounts, issue session token.
  - Errors: expired, used, invalid, rate-limited.

- `GET /account`
  - Returns matter-of-fact account settings, verified email, device sessions, deletion state, and export status. Does not return hidden personality numbers except in export.

- `PATCH /account/settings`
  - Updates audio, captions, reduced motion, narration, timezone, and visit notification preference.

- `POST /account/email-change`
  - Starts verification of new address.

- `POST /account/sessions/:id/revoke`
  - Revokes a device session.

- `POST /account/export`
  - Queues an export and emails a short-lived link to the verified address.

- `POST /account/delete`
  - Soft-deletes account and schedules hard deletion after 30 days.

- `POST /account/delete/cancel`
  - Restores within the 30-day window.

Aviary state and interactions:

- `GET /aviary/bootstrap`
  - Returns the current snapshot, account settings relevant to rendering, server time, local-time mapping, scene asset manifest, and a short-lived event submission token.
  - Snapshot includes bird IDs, names, species, mood labels for internal rendering, perch zones, motion phase, active weather, call schedule hints, scene lighting state, settled state, and canonical revision.
  - Snapshot does not expose numerical personality vectors to the user-facing client state store or UI. If the renderer needs normalized visual parameters such as plumage saturation, return already-derived visual values in a renderer-only payload, not trait labels.

- `GET /aviary/snapshot?since_revision=...`
  - Returns current canonical snapshot, or 304/no-change when appropriate.
  - Called on visibility return, long frame gaps, and low-frequency visible keepalive.

- `POST /aviary/events`
  - Accepts batched interaction events with client IDs.
  - Valid event types are presence ping, listen-in start/end, offers, settle start/undo/complete, and visibility/focus/activity state changes.
  - Server validates bird ownership, offer cooldowns, event ordering windows, session authority, and idempotency.
  - Returns accepted/rejected event IDs and current server time.

- `GET /aviary/notebook`
  - Cursor-paginated read-only notebook entries.

- `GET /aviary/narration`
  - Optional endpoint if narration is server-generated. Returns the next slow-cadence naturalist narration from canonical state. If narration is client-generated, this logic lives in shared deterministic templates and still uses the same state.

Visits:

- `POST /visits/invites`
  - Host creates an invitation for a specific email.
  - Sends a one-time link expiring after 30 days.

- `GET /visits/invites`
  - Host settings view: outstanding invites, recent visits, revocation controls.

- `POST /visits/invites/:id/revoke`
  - Revokes immediately.

- `POST /visits/consume`
  - Visitor consumes token and receives a read-only visit session.

- `GET /visits/:visit_session_id/snapshot`
  - Read-only snapshot of host aviary. Returns revoked/expired surface when no longer available.
  - Does not emit greetings, listen-in, offers, settle, presence, or notebook effects.

Implementation guardrails:

- Reject any route that attempts to write bird personality, mood, notebook, or snapshot state outside the simulation worker.
- Keep visitor APIs physically separate from host interaction APIs so a visitor token cannot call `/aviary/events`.
- Put all public-facing copy through product/system voice review. Product surfaces are naturalist; account/auth/error/accessibility settings are matter-of-fact.

## 5. Simulation engine design

The simulation worker is the behavioral core. It should be deterministic enough to test, but seeded enough that the aviary never feels canned.

Tick cadence:

- Schedule a tick approximately once per minute per aviary.
- If the worker falls behind, process elapsed time in bounded steps rather than one huge catch-up mutation. Cap catch-up work per run and reschedule remainder to protect p99 latency.
- A tick must be idempotent over event consumption. Use event IDs and `consumed_by_tick_at` or a monotonic tick cursor to avoid double-applying drift.

Tick inputs:

- current canonical aviary snapshot;
- birds and hidden personality vectors;
- recent unconsumed interaction events;
- derived presence windows;
- user local timezone;
- settled state;
- active weather or weather generation schedule;
- last notebook entry times and source refs.

Tick outputs:

- updated personality vectors via small additive deltas;
- updated mood, mood timers, perch zone, and motion state;
- call schedule hints and call grammar parameters for the next snapshot interval;
- active/expired weather;
- canonical snapshot revision;
- rare notebook entries;
- operational metrics only: tick duration, events consumed count, errors.

Presence derivation:

- Client emits heartbeat-like presence pings only when all client-side conditions are true: `visibilityState=visible`, window focused, and pointer/key activity within the calibrated window.
- Server does not trust large client totals. It reconstructs presence windows from pings, caps gaps, rejects impossible durations, and stores bounded windows.
- Tune the activity window toward several minutes so quiet watching counts. Initial value: 180 seconds, tested in calibration.
- Tab close and settle both end the current presence window. Settle adds mood quieting but no special drift reward.

Personality drift:

- Represent each trait in a normalized range, for example 0.0 to 1.0, initialized from species-biased seeds.
- Apply drift as a low-pass filter over weekly-scale signal aggregates, not immediate event clicks.
- Presence is the dominant positive input toward expressiveness. Listen-in adds bird-specific weight to social warmth and vocal frequency. Offers add small curiosity/boldness deltas when valid and cooldown-respecting. Plumage saturation drifts upward only with sustained attention.
- No negative drift from absence or neglect. If no presence occurs, do not move traits downward. Quieter behavior after absence should emerge from mood and lack of recent positive signals, not trait punishment.
- Calibration target: regular visits produce measurable numerical movement after about one week and visible/noticeable change after about three weeks.
- Keep drift increments tiny. Example initial calibration: compute daily presence score, smooth with exponential moving average, apply weekly target deltas on the order of 0.01-0.03 per trait for regular use, then tune from simulation fixtures.

Mood transitions:

- Mood enum: `wary`, `content`, `curious`, `drowsy`, `alert`, with room for `settled` as a scene state rather than a bird personality state if easier.
- Mood transition scores combine current mood inertia, time-of-day, active weather, recent interactions, bird-to-bird calls, and personality vector.
- Mood persists across sessions and is updated by ticks while no client is connected.
- Do not reset to neutral on navigation. The first snapshot must reflect the current canonical mood.

Greeting selection:

- On bootstrap or visibility return after absence, the server snapshot should include greeting eligibility and absence-length category.
- Client renders one bird noticing the user within one to two seconds. If several qualify, stagger them with small randomized offsets.
- Selection weights: boldness, social warmth, current mood, recent greeting history, absence length. Warier birds may glance from back perches; bolder birds may step forward or call.
- No textual welcome and no absence-duration text.

Calls and call grammar:

- Each bird has a stable call signature seed and species motif library.
- Server snapshots provide call intent windows and grammar parameters: motif family, mood, intensity, timing density, and chorus participation. The client synthesizes actual audio via WebAudio with deterministic variation from seed plus current time window.
- Recognizability is an explicit test: changing mood or vocal frequency should modify timing/pitch/ornamentation without making a bird sound like a different bird.
- Chorus events emerge from overlapping call windows and bird-to-bird response probabilities. Avoid synchronized on-arrival chorus.

Offers:

- Offer types: seed, song fragment, still pool.
- Enforce per-bird cooldown of a few minutes on the server. Initial value: 4 minutes, tune later.
- Reactions depend on current mood and personality:
  - curious/content birds approach and inspect;
  - wary birds delay or stay back;
  - drowsy birds may ignore;
  - high vocal frequency birds respond more strongly to song fragments.
- Offers are gestures, not feeding mechanics. Never create hunger, inventory, required care, or repeated reward loops.

Settle:

- `settle_start` triggers a slow lighting transition and call quieting.
- Any click or re-engagement within five seconds emits `settle_undo` and reverses the transition.
- If completed, mark the aviary as settled for that client/session until close or active re-engagement. Engine treats settle and tab close as equivalent presence endings, with only a small mood-quieting effect.

Notebook generation:

- Generate entries from notable state changes, rare moments, first-of-week ordering, weather effects, or sustained quiet.
- Enforce sparsity: roughly every few days for a regularly visited aviary, with minimum spacing unless a genuinely rare event occurs.
- Entries are naturalist prose, lowercase, present tense, specific to the birds and scene. They never report user attendance, streaks, numerical trait changes, or event-log wording.
- Use template families with slots sourced from canonical state and event summaries; avoid open-ended generated text unless it is constrained and reviewed, because voice consistency matters more than variety.

## 6. Sync and conflict model

The system avoids client conflict by design.

Canonical rules:

- The server owns all bird identity, personality, mood, notebook, and current snapshot state.
- Clients submit append-only event facts with idempotency keys.
- Simulation consumes events in server order and writes additive deltas.
- Clients render snapshots and may interpolate between revisions, but never write absolute state.

Multi-device behavior:

- Each device uses its own session token.
- If laptop and phone are open at once, both submit their own event streams. Presence should be accepted only from the signed-in account's active device conditions; simultaneous valid presence from two devices can be capped or treated as one account-level presence window to avoid double-counting. Initial call: aggregate overlapping presence windows into a single account-level window per aviary.
- Listen-in and offers from multiple devices are events. The server applies cooldowns and ordering. Rejected events return matter-of-fact UI guidance if the initiating device needs it, but the aviary itself should simply continue.
- Snapshots carry canonical revision. Clients discard stale local interpolation targets when a newer revision arrives.

Conflict/error surfaces:

- Magic-link replay: show a matter-of-fact expired/used link message and offer to request a new link.
- Session timeout: require sign-in again.
- Snapshot load failure: quiet field plus matter-of-fact retry surface outside the scene if needed.
- Revoked visit: visitor gets "This visit is no longer available." Host is not notified.

Data preservation:

- Personality vectors are never rebuilt from event logs as part of normal operation. Event logs support tick consumption and audit/debugging, but the stored vector is canonical.
- Migrations touching bird identity or personality must include snapshot-before/snapshot-after validation and a no-reset invariant test.

## 7. Frontend rendering pipeline

The first screen is the aviary, not a landing page or dashboard. Use a sparse top bar above the scene and no UI chrome inside the scene.

Boot sequence:

1. HTML shell loads with a quiet field background and critical CSS.
2. Bootstrap snapshot and minimal scene renderer initialize.
3. First bird is visible within 500ms on target devices. If data is late, quiet field remains with faint motion cues, not a spinner.
4. Birds start mid-action using snapshot motion phase. No wake-up animation or app-like fade-in.
5. Return greeting occurs within one to two seconds, selected from snapshot guidance.

Scene composition:

- One horizontal responsive scene, no panning, scrolling, or zooming.
- Three logical perch zones: front, middle, back. Perch choice is server-driven and reads as a behavioral signal.
- Foreground/background separation with subtle parallax only.
- Ambient client-only leaf/feather drift at sparse random intervals. These ornaments are not simulation state.
- Local day/night palette computed from timezone and server time. Morning, midday, evening, night states blend gradually. Night remains alive; a nightjar-like species may remain vocally active.
- Rare weather overlays and mood effects come from snapshot state.

Bird rendering:

- Each bird has species silhouette, visual seed, plumage values derived from personality, mood poses, and motion clips/pose sequences.
- Idle motion families: preening, scanning, head tilt, body shuffle, low drowsy perch, wary back-perch scan.
- Use deterministic variation so motion does not loop obviously.
- Keep every bird visible at all supported viewport sizes; responsive layout compresses spacing, not bird visibility.

Interaction UI:

- Top bar icons: account/settings, accessibility settings, field notebook, offer affordance, settle. The PRD names account/settings, accessibility, notebook, and offer; settle is also specified as top-bar-triggered, so include it as a restrained icon affordance.
- Top bar fades nearly transparent after cursor stillness and returns on pointer movement or keyboard activity.
- Listen-in is triggered by click/tap/keyboard focus on a bird. Audio mix ramps gradually; other birds quiet but never mute.
- Offer affordance opens a compact top-bar menu/panel, not an in-scene overlay. Use naturalist wording for offer labels.
- Field notebook opens from the top bar, read-only, with sparse entries and no edit/delete controls.
- Account/auth/settings surfaces use matter-of-fact voice.

Reduced motion:

- Respect `prefers-reduced-motion` by default and allow explicit override in accessibility settings.
- Replace continuous micro-motion with slow cross-fades between still poses.
- Replace flight paths with cross-fades between perch poses.
- Remove leaf/feather drift.
- Keep day/night color shifts, mood changes, calls, captions, notebook, and simulation intact.

Keyboard and focus:

- Tab enters top bar items, then aviary scene.
- Arrow keys move focus between birds.
- Enter toggles listen-in on the focused bird.
- Escape exits listen-in or closes top-bar panels.
- Focus outline must remain visible across bright/dim palettes without turning into intrusive scene chrome.

## 8. Audio pipeline

Use one WebAudio graph per active aviary session, created after browser-permitted user activation if required. Before activation, render gracefully and show captions if enabled; do not block the aviary.

Components:

- species motif library encoded as compact procedural parameters;
- per-bird stable signature seed;
- grammar runtime that produces motif sequences, pitch contours, durations, pauses, and timbral variation;
- per-bird gain/pan/filter nodes;
- ambient chorus mixer;
- listen-in mix controller;
- caption generator from the same call event that creates audio.

Call synthesis:

- Generate short buffers or AudioWorklet output on demand with pooling/reuse.
- Avoid per-call allocation leaks. Reuse nodes where practical.
- Shape calls by mood and vocal-frequency trait-derived render parameters without exposing those traits to UI.
- Mix simultaneous birds as a chorus with small timing offsets and frequency separation to avoid phase artifacts.

Listen-in:

- On engage, ramp focused bird gain upward over a slow interval, for example 800-1500ms.
- Ramp other birds downward to ambient level, not silence.
- On disengage or focus change, ramp back to ambient.
- Submit listen-in start/end events for drift; keep local audio responsive even before server acknowledges, but reconcile rejected/stale events quietly.

Captions:

- Captions are generated from actual call grammar output: "a soft three-note rise," "a low trill, paused, low trill again."
- Position near the calling bird without covering it. In reduced motion or narrow layouts, captions can stack in a dedicated caption lane below the scene if needed to preserve readability.
- Captions are optional except WebAudio fallback, where captions turn on by default.

Fallback:

- If WebAudio is unavailable or fails, no recorded-audio fallback ships. Play in silence with captions on by default and a matter-of-fact accessibility setting note if needed.

## 9. Accessibility surfaces

Accessibility ships with v1 and is treated as part of the product, not a later compliance pass.

Screen-reader narration:

- Provide a live region or dedicated narration surface with slow naturalist prose updates every 30-60 seconds at idle.
- Prioritize user-initiated events: greeting, offer reaction, settle, listen-in changes.
- Never narrate raw state labels like perch indices, trait values, or event logs.
- Voice: lowercase, present tense, specific, no exclamation, no achievement framing.
- Allow user control over narration verbosity/cadence in accessibility settings using matter-of-fact labels.

Semantic model:

- The scene exposes focusable birds by name and species-style description.
- The top bar and settings use ordinary accessible buttons, dialogs, labels, and focus traps.
- The notebook is a readable, scrollable list of entries with dates.
- Offer menus and visit/account settings meet standard keyboard and screen-reader expectations.

Reduced motion:

- Implement as a renderer mode with designed pose cross-fades, not by pausing the normal renderer.
- Test with OS preference and app override.

Contrast:

- All text, icon labels/tooltips, settings, errors, captions, notebook entries, and visible narration pass WCAG AA.
- Validate contrast across day, evening, and night palettes.

Keyboard:

- No pointer-only functionality.
- Listen-in, offers, settle, notebook, settings, invite revocation, account export, and deletion recovery are fully keyboard reachable.

Unsupported browser and account errors:

- Use matter-of-fact language. Do not force naturalist voice into system failure states.

## 10. Privacy, telemetry, and observability

Privacy boundary:

- Per-bird interaction events, personality vectors, mood, notebook sources, and relationship history are stored only to drive that account's own aviary.
- They are not exported to analytics warehouses, training pipelines, recommendation systems, or population dashboards.
- Internal logs use synthetic account IDs and request IDs. Email is encrypted on the account row only and is never a partition key or log identifier.

Allowed aggregate telemetry:

- request counts and latencies by route;
- simulation tick durations and error rates;
- event batch accept/reject counts without event payload content;
- first-bird render timing;
- render frame timing histograms;
- audio context error counts;
- bundle size and asset load timings;
- anonymized session-duration histograms without account dimension;
- synthetic browser check results.

Disallowed telemetry:

- average drift by trait across users;
- per-bird interaction funnels;
- named bird behavior analytics;
- visit popularity rankings;
- user attendance calendars;
- anything that can reconstruct a specific user's relationship with their aviary.

Observability implementation:

- Use structured logs with PII filters and automated tests for forbidden fields.
- Expose service SLO dashboards for API availability, snapshot latency, tick p99, event ingestion errors, email delivery, and client performance.
- Set simulation tick latency p99 alarm at 5 seconds.
- Run synthetic checks from common geographies against bootstrap, first bird render, snapshot refresh, WebAudio initialization, and reduced-motion rendering.

## 11. Performance plan

Budgets:

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No client memory growth over 30 minutes.
- Snapshot payloads in kilobytes, not megabytes.

Implementation tactics:

- Split account settings, accessibility settings, visit management, notebook history beyond first page, and export/delete flows out of the initial bundle.
- Keep scene bootstrap minimal: renderer core, current birds, current motifs, essential controls.
- Use compact procedural species assets and motif parameters. Avoid recorded audio.
- Precompute or edge-cache bootstrap snapshots where safe. At minimum, keep snapshot reads low-latency and avoid blocking first paint on non-critical settings.
- Use requestAnimationFrame scheduling with tab visibility pausing for rendering. Simulation continues server-side.
- Pool audio buffers/nodes and render objects. Avoid creating new allocations for every call, leaf, caption, or pose transition.
- Add CI performance tests: bundle-size gate, first-render synthetic test, 30-minute memory test, frame-time smoke test, and audio allocation test.

## 12. Rollout plan

Phase 0: Prototype/calibration

- Build deterministic simulation fixtures for presence, drift, mood, offers, listen-in, absence, and multi-device overlap.
- Build call grammar prototypes and recognizability tests for two, five, and seven birds.
- Build visual renderer spikes for normal and reduced-motion modes.
- Validate that the quiet-field loading state and first-bird render can meet 500ms.

Phase 1: Internal alpha

- Ship auth, starter aviary creation, canonical snapshot, simulation tick, two bird species, basic renderer, procedural calls, presence pings, listen-in, offers, settle, and sparse notebook.
- Limit aviaries to two birds.
- Instrument operational telemetry only.
- Use staff accounts to tune drift rate, greeting variation, notebook sparsity, and audio uncanniness.

Phase 2: Accessibility-complete alpha

- Add screen-reader narration, call captions, full keyboard navigation, reduced-motion renderer, contrast validation, and WebAudio fallback.
- Treat accessibility bugs as launch blockers.

Phase 3: Private beta

- Add all account settings, session revocation, export, deletion, email change, visit invitations, visit log, invite revocation, and optional visit notifications.
- Keep birds capped at two or three during early beta until audio and performance data support higher counts.
- Test multi-device sessions heavily.

Phase 4: V1 launch

- Enable age-gated third-bird availability for sufficiently old beta aviaries.
- Ramp max birds gradually: start at two for new accounts, allow three after the designed age threshold, and raise the allowed maximum only after recognizability and performance tests pass at each count.
- Keep visits off by default and avoid onboarding prompts for social sharing.

Day-one instrumentation:

- API health, tick p99, snapshot latency, first-bird timing, frame-time histograms, memory test results, audio context errors, event ingestion errors, email delivery status, invite revoke correctness, and account deletion job status.
- No per-bird aggregate dashboards.

## 13. Test strategy

Simulation tests:

- Presence requires visible + focused + recent pointer/key activity.
- Background tabs do not accrue presence.
- Overlapping multi-device presence does not double drift.
- Drift is monotonic toward expressive and never decreases traits on neglect.
- Regular visits show measurable drift after simulated one week and visible-level derived changes after simulated three weeks.
- Absence does not create distress, hunger, death, or negative trait movement.
- Only simulation worker can mutate personality vectors.
- Mood persists across sessions and changes with time/weather/interactions.
- Offer cooldown prevents single-session curiosity saturation.
- Notebook entries are sparse and never mention streaks, visit frequency, or numeric traits.

API tests:

- Magic links expire after 15 minutes and are single-use.
- Session revocation blocks future requests from that token.
- Email change waits for new-address verification.
- Export contains the user's aviary state and is emailed to the verified address.
- Soft deletion can be canceled within 30 days; hard deletion removes all tied records.
- Visit sessions are read-only and cannot submit events.
- Revoked/expired visits terminate at next snapshot pull.
- Client attempts to write personality or mood are rejected.

Frontend tests:

- First frame contains quiet field or bird-in-progress, never spinner.
- No textual welcome appears on return.
- Top bar fades and returns on input.
- Keyboard can reach every interaction.
- Listen-in ramps audio mix and never fully mutes other birds.
- Reduced-motion mode uses cross-fades and removes leaf drift.
- Captions match generated call events.
- Responsive layout keeps every bird visible.
- WCAG AA contrast passes across scene palettes.

Performance tests:

- Bundle-size gate under 2MB gzipped.
- Synthetic first-bird render under 500ms target.
- 30-minute no-memory-growth test.
- 60fps idle on target laptop profile.
- Simulation tick p99 under 5s alarm threshold.

Privacy/security tests:

- Logs and telemetry reject email and per-bird payload fields.
- Synthetic account UUID is used outside the account row.
- Analytics schemas cannot include interaction payloads, personality, mood, or notebook sources.
- Rate limiting covers auth and invite endpoints.
- Token hashes only, no raw tokens persisted.

## 14. Key risks and mitigations

Drift calibration too fast:

- Risk: users feel birds change after a session, turning the product into stat management.
- Mitigation: simulation fixtures across weeks, tiny deltas, internal alpha tuning, no visible numbers, and one-week measurable/three-week visible calibration gate.

Drift calibration too slow:

- Risk: users feel nothing they do matters.
- Mitigation: derive visible changes through mood and greeting behavior while keeping personality slow; use notebook observations to surface rare specific moments without revealing stats.

Sync correctness failure:

- Risk: one device overwrites another, losing personality history.
- Mitigation: server-only vector writes, append-only events, idempotency keys, no last-write-wins state endpoints, migration invariants, overlap presence tests.

Audio uncanniness:

- Risk: calls repeat, phase, or blur into generic ambience.
- Mitigation: procedural grammar, recognizability tests, call signature seeds, chorus mixing tests at seven birds, no recorded fallback.

Accessibility flattening:

- Risk: screen-reader/reduced-motion users receive a state list or static fallback instead of the product.
- Mitigation: ship accessibility surfaces in v1, naturalist narration templates, designed reduced-motion renderer, captions from call grammar, accessibility QA as launch blocker.

First-load failure:

- Risk: spinner or slow first bird breaks the "already alive" conceit.
- Mitigation: edge/bootstrap snapshot, quiet field fallback, minimal initial bundle, first-render performance gate.

Privacy boundary erosion:

- Risk: useful-looking analytics begin aggregating per-bird relationship data.
- Mitigation: schema-level separation, telemetry allowlist, logging filters, review gate for new metrics, no analytics warehouse access to simulation database.

Feature creep toward games/social:

- Risk: achievements, streaks, public discovery, or notifications enter as harmless engagement features.
- Mitigation: encode non-goals in product requirements, design review checklist, no database primitives for those features, no onboarding prompts for visits, notification default off.

Notebook voice drift:

- Risk: generated entries become generic event logs or user-behavior summaries.
- Mitigation: constrained templates, voice tests, sparse entry policy, banned phrase list for achievements/streaks/numeric traits/user attendance.

Renderer performance with seven birds:

- Risk: idle motion, captions, and audio degrade at max count.
- Mitigation: ramp bird cap gradually, load tests at each cap, object pooling, capped ornaments, and recognizability/performance gates before raising allowed counts.

## 15. Engineering milestones

Milestone A: Foundations

- Schema, auth, sessions, account UUID rules, starter aviary creation, two starter species, event ingestion, simulation worker skeleton, snapshot read API.

Milestone B: Living aviary loop

- Server tick updates mood/perch/call hints; client renders bird-in-progress scene; local day/night; presence pings; return greeting; listen-in audio ramp.

Milestone C: Gestures and memory

- Offers with cooldown/reactions; settle/undo; drift deltas; notebook generation; account settings; accessibility settings.

Milestone D: Accessibility and performance hardening

- Screen-reader narration; captions; reduced-motion renderer; full keyboard navigation; contrast; bundle and runtime gates; memory tests.

Milestone E: Account completeness and visits

- Export, deletion, email change, session revocation, invite creation/consume/revoke, visit log, read-only visitor snapshots, optional visit notifications.

Milestone F: Calibration and launch readiness

- Drift tuning, call recognizability, seven-bird stress tests, privacy telemetry audit, copy review, synthetic monitoring, beta ramp controls.

## 16. Definition of done for v1

V1 is ready when:

- a new user can sign in by magic link, meet two starter birds, name them, and see the aviary in motion without an app-like loading sequence;
- the same account on phone and laptop shows one canonical aviary with no merge prompts or personality conflicts;
- presence, listen-in, offers, and settle feed the simulation through append-only events only;
- birds show mood moment-to-moment and slow drift over weeks without visible stats, penalties, or streak surfaces;
- procedural calls are recognizable per bird and remain convincing in chorus;
- field notebook entries are sparse, specific, naturalist, and read-only;
- account export/deletion/session/email flows are complete and matter-of-fact;
- optional visits are read-only, revocable, off by default, and do not affect host bird drift;
- screen-reader narration, captions, reduced motion, keyboard navigation, and contrast are complete;
- initial bundle, first-bird time, idle frame rate, memory, and tick p99 meet budgets;
- telemetry and logs respect the privacy boundary and contain no per-bird relationship analytics;
- no v1 surface implements or hints at gamification, Tamagotchi mechanics, public social networking, native apps, or notification-driven engagement.
