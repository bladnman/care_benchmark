## System-level intent

- Server-canonical simulation is the load-bearing architecture. This shows up in the product summary ("Server is the only writer of personality"), the decision rule ("keeps the server as sole simulation writer"), the API boundary ("Never writes personality vectors or moods"), and the sync model ("Laptop and phone are two snapshot readers"). Clients detect and submit facts; the worker interprets them.
- The product refuses announcement and gamification surfaces. The decision rule says to "refuse announcement/gamification surfaces"; the out-of-scope list excludes "streaks, badges, levels, scores" and "days visited"; snapshot always returns `notebook_unread_hint: false`; the top bar has "No badges, no unread dots"; launch has "No onboarding carousel" and "No 'share with friends' prompt."
- Accessibility is a designed v1 surface, not a later accommodation. The decision rule says to "ship accessibility as a designed surface on day one"; scope includes "naturalist screen-reader narration, designed reduced-motion mode, runtime call captions, WCAG AA chrome, full keyboard path"; rollout says reduced-motion "cannot slip"; accessibility says it is "Same product, different register - not a stripped mode."
- Product voice splits by surface. The product summary says "Voice is naturalist on product surfaces and matter-of-fact on system surfaces"; errors are "matter-of-fact" with "No naturalist phrasing"; visit AT first announces readonly in a matter-of-fact voice, then returns to "naturalist running narration."
- The aviary should feel quietly alive rather than waking up for the user. The frontend forbids "entry animation, spinner, or fade-from-static"; first frame starts with a "quiet field"; birds instantiate at `motion_phase` so they are "mid-cycle"; the definition of done requires "Two birds mid-motion" and "no spinner."
- The relationship model is expressive but not punitive. Drift moves "only up"; "Neglect does not decrement"; the plan implements "quieter, not mistrustful" through `gain`, not trait loss; settle has "zero trait delta"; tab close is "unpunished."
- Presence means actual attention, not an open tab or growth metric. Presence requires visible, focused, and recent pointer/key activity; a background tab produces "zero samples"; the calibration target says 20 minutes/day should not become "24 hours"; observability refuses per-account presence totals in the warehouse.
- Privacy and telemetry minimization are "load-bearing." The privacy architecture encrypts email, keeps simulation DB out of analytics, denies `bird_id`, trait values, offer types, notebook text, and visitor emails in metrics, and says per-bird events exist "solely to drive that account's next ticks."
- Bird identity continuity matters. Birds have stable IDs "never recycled"; species art updates must not "replace birds"; `call_seed` locks motif family "for life"; recognizability depends on stable timbre where mood/personality change "timing and intensity, not identity."
- Social features are intentionally constrained. Visits are opt-in, read-only, revocable, expiring, and silent; "No presence events" are written for visitors; notify is off by default; social creep is called out as a risk where a visit log could become "a notification product."
- Multi-device behavior is convergence through one record, not conflict management. The plan states "There is no CRDT" and "no LWW" for personality; old snapshots cannot overwrite drift because clients "never send vectors"; the only user-facing conflicts are auth/session/load failures.
- Performance budgets protect the product thesis. The first-bird and quiet-field path is tied to no spinner and mid-motion aliveness; budgets are CI gates; first-frame failure is a risk because a spinner/fade "breaks the product thesis."

## Per-feature whys

**Scope and open calls**

- Server-side simulation tick, append-only interaction log, and snapshot pull: keeps the server as "sole simulation writer" and makes events "facts," not client-authored state.
- Browser-only web client for last two major versions: NOT RECOVERABLE FROM PLAN
- Magic-link auth: NOT RECOVERABLE FROM PLAN
- Synthetic account UUID: supports the privacy rule that the account identifier everywhere except the account row is `account_id` UUID, while email is encrypted.
- One aviary per account: NOT RECOVERABLE FROM PLAN
- Two starter birds selected by the system from six species: the server chooses "two different species with complementary priors" so "the first chorus is readable."
- Adoption by age of aviary up to seven: pacing is by "Age of aviary, not attention," with no catalog-style adoption, so new birds are not earned by activity.
- Personality vector, mood enum, procedural call grammar, and mood-shaped idle motion: they drive perch bias, plumage, call rate, mood, and motion while the UI never exposes "personality numbers anywhere."
- Presence accounting using visible, focused, and recent activity: ensures a background tab produces "zero samples" and a 20-minute watcher accumulates about 15-20 minutes, "not 24 hours."
- Multi-device sync as server-canonical state: avoids a merge protocol because phone and laptop are only "snapshot readers" and cannot overwrite drift.
- Visit invitations that are opt-in, read-only, revocable, expiring, and quiet: prevents interaction with the host aviary, avoids badges and host pings by default, and contains "social creep."
- Account export by emailed download link: the plan frames export as "user-initiated only" and keeps the mailer transactional.
- Soft-delete 30 days then hard-delete: NOT RECOVERABLE FROM PLAN
- Operational telemetry only: prevents per-bird or per-account interaction analytics and avoids dashboards that invite engagement features.

**Architecture**

- Web SPA with edge-injected state bootstrap: lets the "first bird" paint without a second RTT when cache is warm.
- API as auth/snapshot/event/account/visit boundary: centralizes user-facing reads and writes while explicitly never writing personality vectors or moods.
- Sim worker claiming due aviaries with row locks: gives one canonical tick writer and allows sharding by `aviary_id` hash without adding "a second source of truth."
- No Kafka in v1, with interaction events in Postgres: preserves Postgres as source of truth and avoids introducing a second source.
- Client/server split where clients never tick: hidden tabs can stop rAF, WebAudio, and presence pings while "Simulation continues on the server."
- Render pipeline with zero interactive DOM inside the canvas except focus proxies: keeps chrome from overlaying badges on birds while preserving keyboard and AT access.
- Privacy architecture with encrypted email, no sim-table analytics replica, and a telemetry denylist: prevents PII leaks, warehouse joins, and relationship reconstruction.

**Data model**

- Revocable sessions listed as device rows: lets settings expose sessions and revoke them by setting `revoked_at`.
- Magic-link consume transaction that creates account, aviary, and two birds together: keeps first-time creation atomic; replay of a consumed token is an error, not another session.
- Persisted `settled_at`/`settled_until` for 15 minutes: lets a visitor or other device see evening if the host just settled.
- Bird IDs that are stable and never recycled: preserves identity continuity and prevents content updates from replacing birds.
- Personality columns updated only by the sim worker: enforces the hard rule that personality is simulation state.
- Append-only `interaction_events`: events are "facts," never commands such as "set boldness to X."
- Consumed event retention for 90 days: supports audit/debug "of that account only" while preventing warehouse copying.
- Read-only notebook entries: keeps the field notebook as observation, not user-authored or behavior-praise content.
- Visit sessions that only update visit timing: keeps visitors from writing presence events or interacting with the host aviary.
- Static, versioned `species` config: NOT RECOVERABLE FROM PLAN
- Owner snapshots include traits but visitor snapshots strip them: owner rendering can run offline, while shared links never leak a numeric "stat sheet."
- Presence events with `active_ms` flushed every 30 seconds and on pagehide: makes drift depend on recorded active time; "A background tab produces zero samples."

**API surface**

- `POST /auth/magic-link` always returns 202: avoids account enumeration.
- Invalid, expired, or used magic links show matter-of-fact HTML: keeps auth errors out of the aviary and out of naturalist voice.
- No "welcome" payload on first-time consume: first load uses "the same snapshot shape as every other load," avoiding a special onboarding surface.
- Snapshot `ETag: tick_version`: supports 304 reads from the monotonic canonical version.
- Snapshot `next_bird_available` without "day 45!" copy: exposes availability without urgency copy.
- Snapshot `notebook_unread_hint: false`: the notebook is "not badged."
- Visit snapshots remove traits, cooldowns, and greeting and add `readonly`: preserves read-only isolation and privacy.
- Event validation, idempotency keys, and server-side cooldowns: make the server authoritative and collapse duplicate submissions.
- Priority tick for interactive events: gives immediate mood/perch/pose/cooldown/lighting reactions while using only a "tiny" drift slice so clicking cannot visibly move traits.
- `rename` writes immediately in the API transaction: names are "user intent, not drift" and "not simulation state."
- Notebook API has no POST: preserves the notebook as read-only.
- Age-gated bird offer appears only when available and without a catalog: makes the new bird feel like "a new bird is at the edge of the scene," not a shop or chooser.
- Visit revoke returns 410 with matter-of-fact body: keeps revocation immediate and system-voiced.
- Optional visit notification sends one matter-of-fact email only on first snapshot: avoids push and avoids mentioning it in onboarding.
- Unsupported browser static HTML: uses the same system register and avoids in-aviary toasts.

**Simulation engine**

- Tick loop with catch-up virtual minutes only for mood, time-of-day, and weather: drift uses only real presence samples and never invents "they would have been watching."
- Monotonic drift function: traits move "only up" and neglect does not decrement, avoiding punishment.
- Weekly calibration targets and fixture tuning: makes changes "measurable in fixtures, not visible session-to-session" and avoids tuning "by gut."
- Separate 28-day expression `gain`: implements "quieter, not mistrustful" because quietness is reversible without lowering traits.
- Settle ending the presence window with zero trait delta: makes settling non-punitive while gently pushing mood toward `drowsy`.
- Mood transitions as a soft attractor: avoids a midnight snap and never forces `content` on tab open.
- Bird-to-bird wary bias and chorus tagging: lets neighbor alarm and overlapping call windows affect audio/narration without client-authored state.
- Call grammar with seed-stable timbre: preserves "Recognizability" because species and `call_seed` keep identity stable while mood changes timing and intensity.
- Perch-zone assignment by mood, boldness, and occupancy: expresses mood/personality while avoiding overlap.
- Weather seeded by `aviary_id` and date: keeps host and visitor in agreement; short rain and wind remain gentle, "Never storms/snow."
- Greeting plan computed at session start: makes return greetings depend on absence and mood, while `greeting_id` prevents replaying a second full greeting on refresh.
- Notebook scorer after mood/drift: emits sparse "noteworthy" naturalist entries and rejects templates that mention the user, streaks, or trait deltas.
- New bird insertion with new UUID and stable row forever: preserves identity; the bird is not reset or recycled.
- Deterministic sim fixture tests: verify drift ranges, zero-presence quieting, commutative device events, API rejection of absolute traits, the seven-bird cap, visitor rejection, and notebook caps.

**Sync model**

- IndexedDB cache for first paint only: supports "quiet-field -> first paint" while giving authority back to the network snapshot.
- Conflict prevention for personality through worker-only writes: prevents stale devices from overwriting morning drift.
- Last-write-wins only for names, settings, and settle: those are user intent or session lighting, not personality.
- Dual-device presence cap: prevents dual-device watching from producing "2x drift."
- Listen-in counting across devices: attention to two birds can count for both; same-bird duplicate attention takes max duration, not sum.
- Visitor keepalives updating only visit sessions: keeps the host snapshot unchanged.
- No sync conflict merge UI: personality "cannot diverge," so only auth/session/load failures are user-facing.

**Frontend rendering**

- Small Svelte or Preact chrome with code-split settings, visits, account, and notebook: supports the bundle and first-bird budgets.
- Canvas2D plus SVG silhouettes for v1 unless the render spike proves WebGL2 is needed: chooses the simpler renderer if it hits 60fps and bundle goals.
- No entry animation, spinner, or fade-from-static: keeps the aviary from visibly waking up.
- Safe stage with letterboxing and compressed narrow layout: all slots remain visible and birds are never cropped.
- Calm naturalist palette, subtle parallax, and low-contrast weather: keeps weather and scene motion "Never assertive."
- Mid-action first frame from quiet field, snapshot/cache, and `motion_phase`: produces birds already in motion with "0 JS heavy" quiet field first.
- AudioContext gesture handling without a modal: captions-ready silence is acceptable until a first pointer/key unsuspends audio.
- Idle micro-motion while visible and rAF cancellation while hidden: keeps the scene alive only when visible and avoids background work.
- Reduced-motion as slow pose cross-fades rather than flight paths: keeps it as "a second art pass," not `animation: none`.
- Top-bar icon chrome that fades and has no badges or labels in the scene: preserves the quiet surface and avoids gamification.
- Offer menu not opened by clicking a bird: keeps offers in chrome rather than turning birds into catalog targets.
- Quiet field during waits and inline chrome errors on failure: avoids circular spinners and messages over birds.

**Audio pipeline**

- WebAudio synthesis from numeric motif libraries: avoids sample files, recorded calls, and large assets.
- Listen-in mix with 1800ms ramps and other birds never muted: focuses one bird while keeping the chorus alive and avoiding hard cuts.
- Independent chorus schedulers with seed-offset LFOs: avoids identical phase and loop stacking.
- Night settle lowering global call rate and gain, with nightjar exempt: preserves the "night-active caller."
- Audio fallback that forces captions on and has no recorded MP3 path: chooses silence plus captions rather than canned audio.
- Buffer hygiene with oscillator pools and disconnects: prevents retained per-call allocations and supports the 30-minute memory gate.
- Captions generated from the scheduled motif sequence: keeps captions aligned to actual audio and in naturalist voice, not internal IDs like `call_03`.

**Accessibility surfaces**

- Screen-reader narration in a polite live region with cadence limiting: provides naturalist observation without spam or assertive interruption.
- Narration that avoids raw moods and trait numbers: prevents product internals from leaking into AT.
- Focused bird observations that are not phrased as "selected": keeps the register naturalist rather than widget-like.
- Reduced-motion mode with still-pose cross-fades and no leaf/feather drift: provides the same product in a different motion register.
- Captions defaulting on when audio is unavailable: preserves call information when WebAudio fails.
- Caption contrast with an adaptive pill: maintains AA in both day and night scenes.
- Keyboard path through top bar and bird focus proxies: makes listen-in, offers, settle, and bird navigation fully keyboardable.
- Visit accessibility with readonly announcement then running narration: combines matter-of-fact system status with the same naturalist narration visitors can receive.

**Performance, observability, and rollout**

- First-paint JS, first-bird, FPS, memory, snapshot, and tick budgets: turn product responsiveness and quiet aliveness into gates.
- CDN quiet field, edge snapshot injection, cached snapshot, progressive feather detail, and deferred chunks: all serve the "TTFB-bird" path.
- Synthetic browser checks and aggregate-only RUM: measure first-bird, FPS, audio errors, and tick lag without account, bird, or trait identifiers.
- Refusal to measure relationship funnels or trait dashboards: avoids building the dashboard that would invite engagement features.
- CI performance tests: keep bundle size, first-bird timing, soak memory, and tick microbench green on main.
- Engineering phases from spine through harden: freeze contracts early, then layer aliveness, gestures, account completeness, and review.
- Reduced-motion in P1 and accessibility review in P4: keeps accessibility from slipping past launch.
- Launching with exactly two birds and disabling age offers for 30 public-launch days: gives support and calibration "a uniform population."
- Shipping all six species at launch: prevents later adoptees from becoming "content updates" that rewrite identity.
- Feature flags as operational, not emotion experiments: avoids experimenting on the relationship surface.
- No onboarding carousel, no share prompt, and no non-transactional emails: keeps launch posture quiet and non-growthy.

**Risks and mitigations**

- Drift calibration mitigations: three-signal presence, caps, `max_tick`, published weekly deltas, and separate gain prevent "Tamagotchi," "screensaver," and over-drift failure modes.
- Sync correctness mitigations: DB role/trigger, API schemas without trait fields, idempotent events, and monotonic `tick_version` prevent client caches and LWW from corrupting traits.
- Audio uncanniness mitigations: seed-stable timbre, jitter, no sample loops, greeting combinatorics, and 1.8s ramps address repeated or phasey calls.
- Accessibility regression mitigations: cadence limiter, night contrast tokens, lint forbidding trait keys in ARIA, AT test, and captions on audio fail prevent visual-only or leaky surfaces.
- First-frame mitigations: quiet-field CSS, cache paint, no ready-pop, CI budgets, and forbidding toast libraries protect the no-spinner thesis.
- Offer/drift saturation mitigations: server-side cooldown, tiny offer deltas, and `log1p` diminishing returns prevent button-mashing curiosity.
- Social creep mitigations: notify off by default, no badge, no public routes, and no global visit ranks prevent visits from becoming a notification product.
- Privacy leakage mitigations: deny email in log fields, separate mailer decrypt path, no sim replica to analytics, and user-initiated export prevent logs and warehouse joins from exposing identity.
- Identity continuity mitigations: immutable `birds.id`, additive `species_id`, no DELETE+INSERT for content updates, and no reset control prevent species art migration from replacing birds.
