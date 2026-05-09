# Pocket Aviary v1 Implementation Plan

## 1. Product Boundary and Scope

Pocket Aviary v1 is a web-only, single-user, single-aviary product centered on a small group of procedural birds that continue living on a server-side simulation clock. The engineering plan should optimize for felt continuity, individual bird recognizability, privacy, accessibility, and restraint rather than feature volume.

V1 includes:

- Browser-only client for modern Chrome, Safari, Firefox, and Edge.
- Email magic-link accounts with one canonical aviary per account.
- Two starter birds per new aviary, with slow age-based growth up to seven birds.
- Stable bird identity, user-editable bird names, species from a small coherent pool, hidden personality vectors, persistent moods, procedural calls, idle micro-motion, and bird-to-bird interaction.
- Server-side simulation tick, append-only interaction events, canonical state snapshots, multi-device sync, account export, account deletion, and session revocation.
- Core interactions: return-greeting, listen-in, offer, settle, field notebook, presence accounting, account/settings, accessibility settings, and optional read-only visit invitations.
- Accessibility surfaces: screen-reader narration, reduced-motion rendering, keyboard navigation, WCAG AA text contrast, call captions, and graceful silence with captions if WebAudio is unavailable.
- Aggregate operational telemetry only: performance, latency, errors, anonymous session-duration histograms, client frame timing, audio-context failures, and simulation-tick health.

V1 explicitly excludes:

- Native apps, native-specific protocols, push notifications, payments, subscriptions, public profiles, public discovery, shared aviaries, household/team accounts, co-presence, chat, comments, follows, leaderboards, rankings, badges, achievements, streaks, levels, XP, scores, visible visit calendars, hunger, death, distress, happiness meters, feeding schedules, user-controlled perches, customizable scenes, multiple aviaries per account, and numerical personality/stat surfaces.

Product voice boundaries:

- Aviary, notebook, captions, narration, offer reactions, and other product surfaces use lowercase, present-tense, specific naturalist prose.
- Sign-in, account, error, conflict, revocation, unsupported-browser, privacy, export, deletion, and accessibility settings use matter-of-fact system prose.
- There is no textual welcome on return. The bird greeting is the welcome surface.

## 2. Architecture Overview

Build as three cooperating surfaces:

1. A browser client that renders the scene, synthesizes audio, detects presence, captures interactions, interpolates snapshots, and presents account/accessibility/notebook/visit UI.
2. A product API that owns authentication, account records, invite flows, snapshot delivery, event ingestion, notebook reads, export, deletion, and privacy-safe telemetry.
3. A simulation service/worker that is the only writer of canonical aviary state, personality vectors, mood state, notebook entries, bird positions, active transition state, and simulation-derived event consumption markers.

Recommended deployment shape:

- Static client assets served from CDN/edge.
- Product API behind a standard web service boundary.
- Background simulation scheduler running approximately once per minute per active/non-deleted aviary, with sharding by synthetic account UUID.
- Relational database for canonical accounts, birds, aviaries, snapshots, invites, sessions, and notebook entries.
- Append-only event table or durable queue-backed event log for client interaction events.
- Object storage or generated-on-demand signed URL for account export JSON.
- Metrics pipeline isolated from per-bird/per-account simulation records.

Core invariant:

- Clients never mutate personality, mood, bird positions, or notebook entries directly. Clients submit events. The simulation tick consumes ordered events and writes canonical state.

## 3. Data Model

Use synthetic UUIDs throughout internal storage, logs, queues, metrics tags, and URLs where possible. Store email only on the account record, encrypted, never as an identifier.

### Accounts

`accounts`

- `id` UUID primary key.
- `encrypted_email`.
- `email_verified_at`.
- `created_at`, `updated_at`.
- `deleted_at`, `hard_delete_after`.
- `settings`: JSON including accessibility preferences, captions default, reduced-motion override, visit notification opt-in, timezone, audio preference.
- `privacy_policy_version_acknowledged`.

`sessions`

- `id` UUID.
- `account_id`.
- `device_label`, `created_at`, `last_seen_at`, `revoked_at`.
- Hashed session token, never raw token.

`magic_links`

- `id` UUID.
- `account_id` nullable until account lookup/creation.
- Hashed token.
- `email_hash_for_rate_limit`.
- `expires_at` set to 15 minutes after issue.
- `consumed_at`.

### Aviaries and Birds

`aviaries`

- `id` UUID.
- `account_id` unique.
- `created_at`.
- `timezone`.
- `bird_count_cap` default 7.
- `settled_state`: enum/metadata for current settled lighting if active.
- `last_simulated_at`.
- `simulation_version`.

`birds`

- `id` UUID stable internal bird identity.
- `aviary_id`.
- `species_id`.
- `name`.
- `adopted_at`.
- `personality`: JSON or typed numeric columns for boldness, social warmth, vocal frequency, plumage saturation, curiosity.
- `mood`: enum such as wary/content/curious/drowsy/alert/settled.
- `mood_updated_at`.
- `current_perch_zone`: front/middle/back plus local perch coordinates.
- `current_motion_state`: preen/scan/call/head_tilt/rest/fly_transition/etc.
- `call_signature_seed`: stable seed for motif variation.
- `visual_seed`: stable seed for species-specific detail.
- `last_offer_at_by_type`: cooldown support.
- `created_at`, `updated_at`.

Personality values are hidden. Do not expose them through user APIs, ARIA labels, settings, notebook text, exports intended for UI display, or debug panels. The account export may include raw values because the PRD explicitly names them, but the export must live in account settings as a private data portability artifact, not as a product surface.

`species`

- Static catalog in code or database migration: approximately six species.
- Silhouette metadata, palette ranges, default motif grammar, motion pose set, night-active flag for the nightjar-like species.
- No rarity field in v1.

### Event Log

`interaction_events`

- `id` UUID or monotonic sequence.
- `account_id`, `aviary_id`.
- `bird_id` nullable depending on event.
- `device_session_id`.
- `type`: presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song_fragment, offer_still_pool, settle_start, settle_undo, settle_complete, adoption_name_set, rename_bird, visit_snapshot_pull, account_setting_changed.
- `occurred_at_client`, `received_at_server`.
- `payload`: bounded JSON, validated per type.
- `processed_at`, `simulation_tick_id`.

Important event rules:

- Presence pings are accepted only from authenticated host sessions, never visitor sessions.
- Visitor events are limited to snapshot pulls and invite/session lifecycle events. They never enter drift calculations.
- Offer events are validated against per-bird cooldown server-side.
- Listen-in duration is derived from start/end events with server receipt bounds to prevent runaway durations.

### Canonical State and Snapshots

`aviary_snapshots`

- `aviary_id`.
- `version` monotonically increasing.
- `generated_at`.
- Compact render state: birds, perches, current mood labels for rendering logic, current transition progress, call schedule window, day/night phase, weather, settled flag, notebook unread count if needed.

Snapshots are small and optimized for first paint. Personality raw values are not included. The client receives derived render parameters sufficient to draw birds and synthesize current calls.

### Notebook

`notebook_entries`

- `id` UUID.
- `aviary_id`.
- `created_at`.
- `observed_at`.
- `text` naturalist prose.
- `source_event_refs` internal nullable.
- `entry_type`: greeting_order, quiet_morning, weather_response, first_recent_offer_response, long_idle_observation, etc.
- `sparsity_key` for rate limiting.

Notebook entries are read-only, sparse, and never generic event logs. Avoid entries about user behavior frequency such as "you visited every day."

### Visits

`visit_invites`

- `id` UUID.
- `host_account_id`, `aviary_id`.
- `visitor_email_encrypted`.
- Hashed one-time token.
- `created_at`, `expires_at` 30 days, `used_at`, `revoked_at`.
- `visitor_session_id` nullable.

`visit_sessions`

- `id` UUID.
- `invite_id`, `host_account_id`, `aviary_id`.
- `started_at`, `last_snapshot_at`, `ended_at`, `revoked_at`.

`visit_log_entries`

- `id`, `host_account_id`, `visitor_email_encrypted`, `visited_at`, `approx_duration_bucket`.

The visit log is account-settings transparency, not a notification feed.

## 4. API Surface

All user-facing APIs must maintain the product's quietness. Return structured errors for clients to render in matter-of-fact voice. Do not include gamified summaries, streak counts, trait numbers, or engagement rankings.

### Authentication and Account

- `POST /auth/magic-link/request`
  - Body: email.
  - Creates or finds account by encrypted email workflow, rate-limits by hashed email, emails 15-minute link.
  - Response always neutral to avoid account enumeration.

- `POST /auth/magic-link/consume`
  - Body: token.
  - Invalidates token on success, creates per-device session token.
  - Returns account summary, session metadata, and whether starter adoption/naming is needed.

- `GET /account`
  - Returns matter-of-fact account settings, sessions, export/deletion status, accessibility settings, visit notification setting.

- `PATCH /account/settings`
  - Updates settings such as reduced motion, captions, audio, timezone, visit notification opt-in.

- `POST /account/email-change/request` and `POST /account/email-change/confirm`
  - New email verification required before switch.

- `POST /account/export`
  - Generates JSON snapshot and emails signed download link to verified address.

- `POST /account/delete`
  - Starts 30-day soft-delete window.

- `POST /account/delete/cancel`
  - Restores soft-deleted account inside window.

- `POST /sessions/{id}/revoke`
  - Revokes a device session.

### Aviary State

- `GET /aviary/snapshot`
  - Authenticated host pull.
  - Returns compact canonical snapshot, snapshot version, server time, local day phase, current weather, birds render state, active calls in near-future window, current settled state, and top-bar UI affordance availability.
  - Does not return personality raw values.

- `GET /aviary/bootstrap`
  - Edge-optimized initial payload for first navigation. Can be embedded in HTML or served as a small low-latency request.
  - Target: enough state to render first bird within 500ms on mid-tier mobile over 4G.

- `POST /aviary/events`
  - Accepts a batch of interaction events.
  - Validates event schema, session ownership, timestamps, cooldowns, and idempotency keys.
  - Returns accepted/rejected event ids plus optional refreshed snapshot version.

- `GET /aviary/notebook?cursor=...`
  - Returns read-only notebook entries.

- `PATCH /birds/{bird_id}/name`
  - Renames a bird without changing identity, mood, species, call seed, or personality.

### Offers

Offers may be represented as event types through `/aviary/events`, but the client also needs a discoverability endpoint:

- `GET /aviary/offers`
  - Returns available offer types and current cooldown availability.
  - No language like rewards or effects. Use naturalist prompts client-side.

Server validation:

- Per-bird cooldown of a few minutes.
- Song fragment selected from a small fixed library.
- Seed and still-pool payloads bounded.

### Visits

- `POST /visits/invites`
  - Host enters visitor email. Creates one-time invite, emails visitor.

- `GET /visits/invites`
  - Host settings view for outstanding/active invites and visit log.

- `POST /visits/invites/{id}/revoke`
  - Immediate revocation.

- `POST /visits/consume`
  - Visitor token consumption. Creates visit session if token is valid, unexpired, and unrevoked.

- `GET /visits/{visit_session_id}/snapshot`
  - Read-only snapshot pull. Returns the host aviary as-is.
  - Does not trigger greeting, presence, drift, listen-in, offers, notebook writes, or host notification by default.
  - If revoked/expired, returns matter-of-fact "visit no longer available" state.

## 5. Simulation Engine

The simulation engine is the behavioral core. It must be deterministic enough to test and varied enough to feel alive.

### Tick Scheduler

- Run a server-side tick roughly once per minute per aviary.
- Tick cadence can be jittered slightly per aviary to avoid thundering herd behavior.
- Tick reads all unprocessed host interaction events since the last tick, current canonical bird state, account timezone, ambient weather schedule, and simulation version.
- Tick writes updated moods, personality deltas, positions, motion states, call schedule seeds, weather effects, and sparse notebook entries in one transaction.
- Tick marks consumed events with `simulation_tick_id`.
- Tick p99 latency alarms at 5 seconds.

Do not tick on the client. Do not recompute personality from history on demand. Do not let multiple devices produce divergent simulations.

### Presence-Time Computation

Client presence detector:

- `document.visibilityState === "visible"`.
- Window has focus.
- Pointermove or keypress occurred within calibrated activity window, initially 3-5 minutes.
- The client emits bounded presence pings only while all conditions hold.
- Pings stop on hidden, blur, settle, tab close/pagehide, or stale activity window.

Server presence validator:

- Accepts pings from authenticated host sessions only.
- Deduplicates/idempotently clamps pings by session and time bucket.
- Caps maximum credited presence per wall-clock window to prevent runaway timers.
- Ends presence on settle or absence of valid pings.

Presence is the dominant drift input. "Tab open" alone never counts.

### Drift Function

Implement drift as a slow low-pass filter over daily/weekly aggregates:

- Maintain rolling per-bird drift inputs: host presence-time, listen-in duration, valid offer attempts/acceptances, and settle events.
- Compute small additive deltas per tick or daily consolidation.
- Clamp values to the allowed normalized range.
- Move traits monotonically toward expressive under positive presence; do not move traits down on neglect.
- Translate less recent presence into quieter ambient expression without negative personality drift.

Trait tendencies:

- Presence-time: broad positive expressive pressure across boldness, social warmth, vocal frequency, plumage saturation, and curiosity, weighted by bird baseline and current mood.
- Listen-in: stronger positive pressure on the focused bird's social warmth and vocal frequency.
- Offer near a bird: small boldness pressure.
- Accepted offer: small curiosity pressure.
- Settle: mood-quieting only; no meaningful drift vector beyond ending presence.

Calibration targets:

- Instrument-detectable personality movement after about one week of regular visits.
- User-visible felt change after about three weeks.
- No single session produces visible trait movement.

Testing approach:

- Golden simulations for one day, one week, three weeks.
- Property tests that neglect never decreases personality values.
- Regression tests that multiple devices cannot overwrite drift.
- Tests that visitor sessions never contribute drift.

### Mood System

Mood is an enumerated fast-timescale state, likely: wary, content, curious, drowsy, alert, settled. It persists across sessions and is advanced by tick.

Inputs:

- Recent host interactions.
- Time of day in account timezone.
- Ambient weather.
- Bird-to-bird calls and nearby moods.
- Personality vector.
- Absence length for greeting selection.

Rules:

- Mood does not reset to neutral on tab open.
- Dusk trends drowsy/settled.
- Early morning trends alert/content.
- Rain dampens vocal frequency and may quiet birds.
- Wind can raise alert/wary probability.
- High boldness reduces wary transitions.
- High curiosity increases offer investigation.
- High vocal frequency increases chorus participation.

### Return-Greeting

On snapshot generation for a returning host session, choose at most one primary greeting bird in the first one to two seconds, with possible staggered secondary response.

Inputs:

- Absence length since last host-visible session.
- Bird boldness, social warmth, mood, current perch, time of day.
- Recent greeting history to avoid identical repeated patterns.

Outputs:

- Render instruction: glance, head tilt, short step forward, quiet two-note call, longer call, or call-and-response.
- Audio call event if applicable.
- Screen-reader narration event if narration is enabled.
- Possible sparse notebook candidate if the greeting is noteworthy, but never every session.

No toast, banner, "welcome back," absence counter, or textual return message.

### Calls and Audio Grammar

Each species has a motif library. Each bird has a stable call signature seed and personality-shaped variation.

Runtime call generation:

- Client receives call grammar id, bird seed, mood, vocal-frequency-derived probability, current call schedule window, and mix parameters.
- WebAudio synthesizes motifs with pitch, envelope, timing, small ornamentation, and inter-call spacing.
- Two birds calling together produce distinct generated calls mixed at runtime.
- Listen-in gradually raises the focused bird and lowers others to ambient, never to silence.
- Disengage ramps back slowly.

Recognition requirement:

- A bird's call signature remains recognizable across mood and personality drift.
- Seven birds is the cap because beyond that the chorus risks becoming undifferentiated.

Fallback:

- If WebAudio is unavailable, play in silence with captions on by default.
- No recorded-audio fallback.

### Notebook Generation

Notebook entries should be generated by a rule-based observation composer in v1, with templates constrained by naturalist voice and fed by real simulation state.

Candidate triggers:

- Unusual greeting order, such as one bird greeting first for the first time this week.
- Long quiet stretch.
- Weather affecting bird behavior.
- A rare offer reaction.
- A bird spending unusual time on a perch zone compared with its recent baseline.
- Bird-to-bird call response.

Sparsity:

- Approximately one entry every few days for regular use.
- More frequent only when genuinely notable.
- Hard cap per aviary per day/week.

Content rules:

- Lowercase, present-tense, specific.
- Names and observed behavior, not metrics.
- Never "achievement," "streak," "visited," "score," "level," "frequency changed by X," or user-judgment language.

## 6. Frontend Rendering Pipeline

### Client Framework and Boundaries

Use a modern browser stack with:

- A small initial JS bundle, aggressively code-split settings/notebook/visit flows.
- Rendering layer separated from simulation logic. Client renders snapshot-derived state and local interpolation only.
- Dedicated modules for scene render, bird pose/motion, audio synthesis, presence detection, event batching, accessibility surfaces, account/settings, and telemetry.

Canvas/WebGL/SVG choice should be validated by prototype against 2MB gzipped initial bundle, 500ms first bird, 60fps idle on a five-year-old laptop, and reduced-motion needs. A hybrid approach is acceptable: SVG or lightweight Canvas for birds/perches, CSS for day/night color, WebAudio for calls, DOM for top bar and accessibility surfaces. Avoid heavy engines unless they demonstrably fit the budget.

### Bootstrap and Loading

Target the first rendered state as the aviary, not a loading sequence.

Implementation:

- Serve HTML shell with embedded or edge-fetched bootstrap snapshot.
- Paint quiet field immediately if snapshot is delayed.
- Render first bird as soon as minimal species/pose data and snapshot arrive.
- Do not block first bird on notebook, settings, visit log, account export code, or full species asset set.
- No spinner. No "loading aviary" text inside the scene. If matter-of-fact loading text is ever required for assistive tech, keep it outside the visual aviary experience and ensure it does not become a product announcement.

### Scene Composition

The aviary is one horizontal scene:

- No panning, scrolling, zooming, or user-controlled camera.
- Three perch zones: front, middle, back.
- Birds always remain visible at supported viewport sizes.
- Responsive layout compresses or expands perch spacing without cropping birds.
- Top bar above the scene, not inside it.
- Top bar contains account/settings, accessibility settings, notebook, and offer affordance.
- Top bar fades nearly transparent after cursor stillness and returns on pointer/keyboard activity.

Day/night:

- Use account timezone.
- Gradual color transitions for morning, midday, evening, night.
- Night is quiet, not dead; nightjar-like species may remain active.

Weather:

- Rare, subtle rain and wind.
- Client renders weather from server state and local ornamentation.
- Weather has small mood/audio effects handled by simulation.

Ambient motion:

- Birds never read as paused.
- Client-side leaves/feathers drift at slow random intervals; these do not require canonical state.
- Parallax is subtle and bounded.

### Bird Rendering

Each bird renderer needs:

- Species silhouette and palette.
- Stable visual seed.
- Current perch zone and interpolated local position.
- Mood-shaped pose/motion set.
- Personality-derived render hints such as bolder posture or richer plumage, without exposing numbers.
- Reduced-motion pose sequence equivalents.

Motion examples:

- Wary: back perch, scanning, smaller movements.
- Content: preening, relaxed posture.
- Curious: head tilts, watching offers/sounds.
- Drowsy: low body, fluffed feathers, slower transitions.
- Alert: upright, responsive calls, quicker glance.

Do not add labels, hover tooltips, mood badges, status icons, or inline UI inside the aviary.

### Interaction UI

Listen-in:

- Pointer/tap on bird or keyboard focus plus Enter.
- Focus indicators visible but soft.
- Click focused bird, empty space, focus another bird, or Escape/focus away to disengage.
- Audio ramps, visual focus can be subtle. No "selected" label.

Offer:

- Top-bar offer affordance opens small menu for seed, song fragment, still pool.
- Offer effects render in-scene as gestures, not rewards.
- Cooldowns prevent button-mashing; copy should avoid punitive language.

Settle:

- Top-bar gesture.
- Slow evening lighting transition and quieter calls.
- Any click in the aviary within five seconds undoes accidental settle.
- Closing tab without settle is equivalent at engine level and never criticized.

Notebook:

- Opens from icon in top bar.
- Read-only, scrollable, sparse entries.
- Naturalist voice.
- No edit/delete/annotation controls.

Settings/account/accessibility:

- Matter-of-fact voice.
- Keyboard complete.
- Manage sessions, export, deletion, email change, privacy policy, visit invites/log, accessibility preferences.

## 7. Accessibility Plan

Accessibility must ship in v1 and must be built from the same canonical state as the default experience.

### Screen-Reader Narration

Create a narration composer that converts snapshot state into naturalist prose:

- Idle cadence: one update every 30-60 seconds.
- Priority updates: return-greeting, offer reaction, settle, notable call, state change caused by user.
- Slow queue to avoid overwhelming screen readers.
- Lowercase, present-tense, specific prose.
- No raw state labels like "mood: content" or "perch 2."

Use ARIA live regions carefully:

- Polite live region for idle narration.
- Short assertive or prioritized polite region only for user-initiated events if testing shows it is needed.
- Provide controls in accessibility settings to pause narration.

### Reduced-Motion Mode

Trigger from `prefers-reduced-motion` or explicit setting.

Reduced-motion renderer:

- Cross-fades between still poses instead of continuous micro-animation.
- Flight paths become cross-fades between perches.
- Removes ambient leaf/feather drift.
- Slows day/evening transitions.
- Keeps calls, captions, notebook, mood changes, drift, and field observations.

This is not a static fallback. It needs designed poses for every mood and species.

### Captions

Call captions:

- Generated from actual procedural call grammar.
- Short naturalist descriptions near the calling bird.
- Fade in/out with calls.
- WCAG AA contrast.
- Available when audio is off, WebAudio unavailable, user opts in, or system detects audio failure.

Examples: "a soft three-note rise"; "a low trill, paused, low trill again."

### Keyboard

Keyboard flow:

- Tab through top bar controls.
- Tab enters aviary bird focus group.
- Arrow keys move between birds.
- Enter toggles listen-in.
- Escape exits listen-in or closes menus.
- Offer menu fully navigable.
- Settle reachable from top bar.
- Settings, notebook, visit management, export, deletion, and accessibility settings complete by keyboard.

### Contrast and Unsupported Browsers

- All text surfaces pass WCAG AA.
- Unsupported browsers receive a matter-of-fact page explaining supported browser versions.
- Avoid compatibility shims that inflate the initial bundle and compromise first-bird performance.

## 8. Performance and Observability

### Budgets

- Initial JS bundle under 2MB gzipped.
- First bird visible under 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a five-year-old mid-range laptop.
- No memory growth during a 30-minute session.
- Simulation tick p99 latency alarm above 5 seconds.
- Snapshot payloads in kilobytes.

### Client Performance Strategy

- Route-level and component-level code splitting.
- Lazy-load account settings, accessibility settings, visit flows, export/deletion, and historical notebook pagination.
- Preload only starter species assets needed for current snapshot.
- Use compact procedural visual definitions where possible.
- Reuse WebAudio nodes/buffers; avoid per-call allocations.
- Bound animation objects and cleanup on visibility hide.
- Stop rendering when tab is hidden; simulation continues server-side.
- Resume with fresh snapshot on visibility change or long frame gap.

### Observability

Allowed aggregate metrics:

- Request count, API latency, error rates.
- Snapshot size and delivery latency.
- First-bird-render timing.
- Render-frame timing.
- WebAudio context errors.
- Caption fallback activation count.
- Simulation tick latency and failure count.
- Anonymous session-duration histograms with no account dimension.
- Memory growth test results from synthetic runs.

Disallowed:

- Per-bird interaction analytics.
- Per-account bird state dashboards.
- Average drift across accounts.
- Social ranking metrics.
- Anything that can reconstruct a user's relationship with their aviary.

Synthetic checks:

- Automated browsers from common geographies.
- Mid-tier mobile emulation over 4G.
- 30-minute memory-growth run.
- Reduced-motion visual smoke test.
- WebAudio unavailable fallback test.
- Keyboard-only walkthrough.

## 9. Privacy, Security, and Data Lifecycle

Security and privacy are architectural requirements, not policy text alone.

Privacy rules:

- Email encrypted on account record only.
- Synthetic UUID everywhere else.
- Per-bird and per-account interaction events stored only to drive that user's simulation.
- No per-bird data in analytics warehouse.
- No ML/model training on per-bird interaction state.
- Visit sessions do not shape host simulation.

Magic links:

- Expire after 15 minutes.
- Single-use, invalidated immediately.
- Rate-limited per hashed email.
- Neutral request response to avoid enumeration.

Sessions:

- Per-device session tokens.
- Hash tokens at rest.
- Revocable from settings.
- Matter-of-fact timeout/re-auth surfaces.

Deletion:

- Soft-delete immediately, recoverable for 30 days.
- Hard-delete all account, birds, vectors, notebook, telemetry linkage, invites, visit logs, and interaction records after window.
- Scheduled hard-delete job with audit logs that use UUIDs only.

Export:

- On-demand JSON snapshot emailed to verified address as signed expiring link.
- Include birds, names, current personality vectors, current moods, notebook entries, and account settings as specified.
- Keep export quiet in account settings; do not market it as a feature loop.

## 10. Rollout Plan

### Phase A: Foundations

- Account model, magic link auth, synthetic UUID discipline.
- Database schema for accounts, aviaries, birds, events, snapshots, notebook, invites.
- Basic API shell and session management.
- Simulation tick skeleton with canonical state transaction.
- Minimal client bootstrap and snapshot render path.
- Privacy-safe metrics pipeline.

Exit criteria:

- New account can sign in, receive two starter birds, and load canonical snapshot on two devices.
- Client cannot write personality/mood directly.
- Emails never appear in logs or internal IDs.

### Phase B: Bird Engine Alpha

- Species pool.
- Personality vector storage.
- Mood transitions.
- Presence event ingestion.
- Drift low-pass implementation.
- Return-greeting selection.
- Perch selection.
- Bird-to-bird call response model.
- Golden simulation tests.

Exit criteria:

- One-week and three-week calibration simulations hit intended instrument/user-visible bands.
- Neglect never decreases personality values.
- Mood persists across session boundaries.

### Phase C: Scene, Audio, and Core Interactions

- Horizontal responsive scene.
- Bird renderer and mood-shaped idle motion.
- Day/night and subtle weather.
- Procedural WebAudio call synthesis.
- Listen-in ramping.
- Offer interactions with cooldown.
- Settle with five-second undo.
- Quiet field loading state.

Exit criteria:

- First bird visible under 500ms in benchmark profile.
- 60fps idle on target laptop.
- Listen-in never hard-mutes other birds.
- WebAudio unavailable path enables captions by default.

### Phase D: Notebook and Voice Hardening

- Notebook observation composer.
- Sparse entry rules.
- Naturalist prose lint/test fixtures.
- Matter-of-fact system copy inventory.
- Regression tests for forbidden terms and gamification surfaces.

Exit criteria:

- Notebook entries read as observations, not logs.
- No welcome toast/banner or streak/score language exists in UI strings.

### Phase E: Accessibility Complete

- Screen-reader narration composer and live-region tuning.
- Reduced-motion renderer.
- Call captions from procedural grammar.
- Keyboard navigation and focus treatment.
- WCAG AA verification across top bar, settings, captions, errors.

Exit criteria:

- Keyboard-only user can complete core session, offer, listen-in, settle, notebook, settings.
- Reduced-motion mode remains alive and designed.
- Screen-reader narration is slow, specific, and naturalist.

### Phase F: Visits and Account Controls

- Invite creation, email delivery, token consumption.
- Read-only visitor snapshot.
- Revocation and expiration.
- Visit log in settings.
- Optional visit notification toggle off by default.
- Export, deletion, email change, session revocation.

Exit criteria:

- Visitor cannot trigger greeting, offers, listen-in, settle, notebook writes, presence, or drift.
- Revocation terminates visitor on next snapshot pull.
- Visit log is visible on demand with no badges or push.

### Phase G: Private Beta and Ramp

- Start with two birds only for all accounts.
- Enable account-age-based third-bird availability after engine validation, not visit count.
- Ramp bird cap gradually in cohorts: two, then three, then five, then seven.
- Monitor aggregate performance, tick health, support issues, accessibility regressions, and audio fallback rates.
- Do not instrument per-bird engagement analytics.

Launch readiness:

- Performance budgets met.
- Privacy boundary reviewed.
- Accessibility test plan passed.
- Simulation calibration reviewed against golden scenarios.
- Copy audit confirms non-goals are preserved.

## 11. Testing Strategy

### Unit and Property Tests

- Presence detector truth table: visible/focus/activity must all hold.
- Presence stale activity window behavior.
- Drift monotonicity under neglect.
- Client events cannot write canonical state.
- Offer cooldown enforcement.
- Listen-in start/end duration clamping.
- Magic link expiry and single-use.
- Invite expiry/revocation.
- Synthetic UUID use in logs/metrics helpers.
- Notebook prose template constraints.
- Caption generation from call grammar.

### Integration Tests

- Two-device same-account flow: laptop interaction, phone snapshot, no conflict or lost drift.
- In-flight overlapping sessions: ordered events, additive server deltas, no last-write-wins.
- Visitor flow: read-only snapshot, no drift, revocation matter-of-fact surface.
- Account deletion and recovery inside 30 days; hard delete after window.
- Export generation includes specified account data.
- WebAudio unavailable fallback.
- Unsupported browser surface.

### Simulation Tests

- Regular visitor for one week produces measurable drift.
- Regular visitor for three weeks produces visible derived render/call differences.
- Two-week absence produces quieter ambient expression but no negative personality drift.
- Rain/weather dampens calls temporarily.
- Dusk moves likely moods toward drowsy/settled.
- High-boldness bird tends toward front perch/greeting.
- High-curiosity bird investigates offers more often.
- Seven-bird chorus remains individually inspectable in test mix scenarios.

### Frontend and Accessibility Tests

- First bird timing under throttled network/device.
- 30-minute memory test.
- 60fps idle target.
- No bird cropped at supported viewport extremes.
- Top bar fades and returns.
- Keyboard-only core flows.
- Screen-reader narration cadence.
- Reduced-motion snapshots.
- Caption contrast across day/night/weather.
- No text overlap in settings/notebook/offer menus.

### Copy and Non-Goal Tests

Maintain a forbidden-surface audit for user-visible strings and routes:

- welcome back, streak, score, achievement, badge, level, XP, leaderboard, ranking, hunger, died, sad because you left, public profile, follow, comments, discover aviaries, birds adopted counter.

This should be a guardrail, not the only review; product review still validates tone.

## 12. Key Risks and Mitigations

### Drift Calibration Too Fast or Too Slow

Risk: Birds feel like stats that move on click, or like screensavers that never change.

Mitigation:

- Golden simulations for one week and three weeks.
- Feature flags for drift coefficients.
- Internal visual/audio diff tools that show derived changes without exposing numbers to users.
- Beta cohort review focused on felt continuity, not engagement.

### Presence Accounting Corruption

Risk: Background tabs inflate drift and silently break the product's core promise.

Mitigation:

- Conjunctive client detector.
- Server-side caps and dedupe.
- Tests for visibility/focus/activity combinations.
- Synthetic sessions that leave tabs open without focus and verify no presence accrual.

### Sync Correctness and Lost Drift

Risk: Multiple devices overwrite personality state.

Mitigation:

- Server tick is the only writer.
- Event log is append-only and consumed in order.
- No absolute trait writes from clients.
- Transactional tick updates with event processed markers.
- Integration tests for overlapping sessions.

### Audio Uncanniness

Risk: Calls feel looped, harsh, repetitive, or blurred in chorus.

Mitigation:

- Procedural motif grammar with stable per-bird signatures.
- Listening tests for two, three, five, and seven birds.
- Gradual listen-in mix ramps.
- No recorded loops.
- Caption fallback rather than canned audio fallback.

### Accessibility Regression Into Fallback Product

Risk: Reduced-motion or screen-reader users get a flattened product.

Mitigation:

- Accessibility work in core milestones.
- Separate reduced-motion pose design.
- Naturalist narration composer.
- Accessibility acceptance tests before launch.
- Include accessibility modes in design and QA reviews, not as later compliance pass.

### Privacy Boundary Erosion

Risk: Per-bird interaction data enters analytics or ML pipelines.

Mitigation:

- Separate operational telemetry schema.
- No account dimension on session histograms.
- Static checks for PII in metric tags/log fields.
- Data access review for simulation tables.
- Explicit dashboard non-goals: no average drift or bird interaction analytics.

### Product Surface Drift Toward Gamification

Risk: Helpful contributors add streaks, counters, badges, or engagement loops.

Mitigation:

- Forbidden string/route audits.
- Product review checklist for every new surface.
- Keep notebook focused on aviary observations, never user behavior frequency.
- Avoid visible counts even where technically available.

### Performance Budget Misses

Risk: Bundle or rendering weight compromises "already in motion" first frame.

Mitigation:

- Performance budget enforced in CI.
- Early prototype of render/audio approach.
- Edge bootstrap snapshot.
- Lazy settings/notebook/visit code.
- Synthetic first-bird timing checks from the start.

## 13. Engineering Decision Log for Ambiguities

- Use a rule-based simulation and prose-generation system for v1 rather than generative AI. The PRD requires specificity and privacy, and per-bird interaction data must not be used for model training or broad analytics. A deterministic composer is easier to audit, tune, and keep sparse.
- Store raw personality values server-side and include them only in account export because the PRD explicitly requires export to contain them. Do not expose them in the product UI, API responses used for rendering, accessibility labels, notebook, or telemetry.
- Treat account-age-based bird additions as a post-launch v1 ramp, not onboarding. Two birds at start remains fixed.
- Prefer edge-delivered bootstrap snapshots over client-only reconstruction, because server continuity is the core architecture and first-bird timing depends on not waiting for multiple API calls.
- Keep visit notifications as an account setting off by default, never onboarding, never push by default, and never represented as a badge on the main aviary.

## 14. Definition of Done for V1

V1 is done when:

- A new user can sign in by magic link, meet two named starter birds, and return later on another device to the same canonical aviary.
- The first visible frame feels like the aviary was already running.
- Presence is counted only when visible, focused, and recently active.
- The server tick is the only writer of personality and mood.
- Birds drift measurably after about one week and visibly after about three weeks without punishing absence.
- Listen-in, offer, settle, field notebook, screen-reader narration, reduced-motion mode, captions, visits, export, deletion, and session revocation all work.
- The product has no streaks, scores, levels, badges, push notifications, welcome toasts, public discovery, co-presence, visible personality numbers, hunger/death/distress, or native-app dependency.
- Performance, accessibility, privacy, sync, and copy audits pass with automated coverage and product review.
