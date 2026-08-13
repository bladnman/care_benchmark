# Pocket Aviary - phase-2A reconstruction

## System-level intent

1. Calm growth without pressure or game loops. This shows up in "Gamification of any flavor" being out of v1, in age-gated birds being "not engagement" and "not visit-gated," and in repeated bans on streaks, rewards, "you earned this," and "you've been gone N days."

2. One canonical simulation record, with one writer of personality. The architecture names the load-bearing rule as "one writer of personality"; the client "never ticks," "never compute[s] drift," and "Clients cannot PATCH personality." Sync correctness is framed around making conflict on personality "structurally impossible."

3. Personality should be felt through behavior, not exposed as numbers. The plan forbids "personality numbers anywhere a user can see them," sends only derived "effects" and "4-bucket quantizations," and says "Do not send the floats."

4. Presence is gentle attention, not extracted engagement. Presence uses "visibility ∧ focus ∧ recent pointer/key," the activity window leans longer because "watching without moving is the product," and the drift model is "Ambient-not-negative": neglect does not decrement traits or create distress.

5. The aviary should feel already alive. The first frame avoids a spinner, logo sting, and fade-from-static; birds appear "mid-action"; idle motion is "never paused-looking"; and the final line calls the product "a window that notices the user."

6. Privacy is a structural boundary, not just policy copy. The plan splits `account-db` and `sim-db`, forbids warehouse access to `sim-db`, says email appears only in `account-db`, and limits telemetry to "aggregate only" with "No account_id, no bird ids, no event kinds."

7. Visits are ambient and read-only, not social-network features. Invitations are "off by default," per-invite, revocable, read-only, and have no events endpoint; the visit log has "No badge, no push"; social profiles, follows, comments, chat, co-presence, and leaderboards are out.

8. Accessibility is a designed launch surface. The plan says "Accessibility is a designed product surface and ships with v1"; reduced motion is "a designed renderer, not animation: none"; captions, narration, keyboard paths, and contrast are launch blockers.

9. Product voice is split between naturalist prose and matter-of-fact system copy. Aviary, notebook, narration, captions, and offer labels use `naturalist`; auth, errors, settings, identity, and visit-unavailable use `system`; "Naturalist voice is forbidden" on system routes.

10. Sound and motion should be procedural, recognizable, and non-canned. Calls come from species grammar, seed, and WebAudio; no sample bank or recorded-audio fallback is allowed; greetings include session-seeded variation so they are "never identical."

11. The scene is local-time and continuous. Timezone is captured and updated so day/night and mood use a stored local zone, not UTC; day/night and weather transitions are continuous; the night settled pose is a lighting flag plus `drowsy`, keeping mood and lighting orthogonal.

12. Performance is part of the product feel. Quiet-field first paint, the catch-up cap, "Time to first bird < 500 ms," bundle budgets, renderer suspension, and "do not wait on AudioContext" all protect the calm first impression and the "already running" feel.

## Per-feature whys

### 1. Scope and decisions

- Browser-only SPA + API: NOT RECOVERABLE FROM PLAN
- Single-user accounts and one canonical aviary per account: NOT RECOVERABLE FROM PLAN
- Email magic-link auth and session policy: the plan calls 90-day idle sessions, revocation, and 15-minute single-use magic links "Calm, low-friction, revocable."
- Two starter birds: the rollout says all new accounts get two birds and "Do not start anyone at 3+ 'to show richness,'" keeping the first aviary modest.
- Hard cap of seven birds: NOT RECOVERABLE FROM PLAN
- Additional birds by aviary age: the rationale is "Age-only; months-to-year cadence; not visit-gated," and the UI must not frame adoption as "you earned this."
- Server-side simulation tick and client snapshot rendering: the rationale is the "one writer of personality" rule; the client renders snapshots and interpolates but "never ticks."
- Personality vectors, mood, calls, motion, and chorus: the plan wants hidden vectors to become derived effects - perch preference, call rate, plumage tier, mood-shaped idle motion - without exposing raw scalars.
- Presence accounting: the conjunction "visibility ∧ focus ∧ recent pointer/key" makes presence the "dominant drift input" while rejecting a plain open-tab heartbeat.
- Presence pings and activity window: "watching without moving is the product," but 20-second credit pings and server-side limits prevent "tab open overnight" inflation.
- Return-greeting: it is a session-open feature so the aviary can notice returns without a welcome toast, calendar, or "you've been gone N days."
- Listen-in: the mix raises the focused bird while others remain audible, so focus does not turn the aviary into a single isolated sound.
- Offers and cooldown: the 4-minute per-bird, per-kind cooldown is "long enough to stop curiosity saturation in one sitting."
- Settle with undo: settle has "zero drift terms"; it ends the presence window, sets `settled=true`, eases lighting and call gain, and can be canceled inside the 5-second window.
- Field notebook: generation targets "~1 entry / 3-5 days" and is "Sparse by construction"; read-only prose avoids editing, stats, and visit-frequency copy.
- Visit invitations: off-by-default, per-invite email, revocation, read-only rendering, unused expiry, and no events endpoint keep visits ambient rather than social-network surfaces.
- Visit-email toggle: email-only notifications exist because "Product is not a notification surface" and email is "the only existing outbound channel"; default off keeps it quiet.
- Accessibility in v1: the plan says reduced motion, captions, narration, WCAG AA chrome, and keyboard paths "ship with v1" and are not later-train flags.
- Account export: export includes personality vectors because it is "the user's copy," but is emailed as a short-lived download link so numbers are not accidentally rendered in the UI.
- Session list and revoke: session revocation supports the "revocable" account posture named in D14.
- Email change with verify-before-commit: NOT RECOVERABLE FROM PLAN
- Soft-delete and hard-delete: the restore screen keeps soft-deleted sign-in simple, and after 30 days the hard-delete job wipes account-db, sim-db, mail-log PII, and export objects.
- Aggregate operational telemetry: the rationale is that metrics must not reconstruct "a relationship"; RUM is aggregate only and excludes account, bird, and event dimensions.
- Native apps out of v1: the plan says not to design protocols or the data model around "native-client constraints."
- Gamification out of v1: streaks, scores, badges, levels, calendars, counters, XP, ranks, and milestone celebrations are rejected to avoid reward framing.
- Tamagotchi mechanics out of v1: death, hunger, distress, decaying happiness, and negative neglect drift are rejected so quiet absence does not punish the user.
- Social network surfaces out of v1: profiles, follows, discovery, comments, chat, avatars, co-presence, leaderboards, and mutual visits are rejected so visits do not become a social graph.
- Payments, multi-aviary accounts, shared aviaries, customizable scenes, push/SMS, SSO/passwords, and recorded-audio fallback: NOT RECOVERABLE FROM PLAN
- Personality numbers nowhere in production UI: the rationale is that hidden scalars should not become a stats panel, debug overlay, or ARIA-exposed vector.
- Mood enum and settled lighting flag: the rationale is "Matches PRD examples" and "keeps mood and lighting orthogonal."
- Species pool: the rationale is a "Coherent backyard set" plus "one nocturnal signature" through the nightjar-analogue.
- Default name suggestions and `bird_id` identity: the rationale is "Names never replace identity."
- Quiet-field first paint: the rationale is to protect "already in motion" without a spinner.
- Timezone capture and update: the rationale is a "Local-time aviary" where day/night and mood-of-day use the stored zone, not server UTC.

### 2. Architecture, data model, and API surface

- Four deployable units: the plan says to keep services small, with "one writer of personality" as the load-bearing rule.
- Hard pipeline split: `ops-metrics` and any warehouse cannot read `sim-db` so operational measurement stays outside per-bird state.
- UUID account identifiers outside the PII vault: every non-email table, log field, queue key, and metric label uses `account_id` UUID because `account-db` is the only email storage.
- Client/server split for personality and mood: the client may read derived effects but never vectors or drift logic, preserving server authority.
- Client WebAudio from snapshot and grammar seed: audio is synthesized from state and "Never persist[s] audio."
- Visit presence not recorded: visit mode is "Render-only" and must not create presence or drift signals.
- Render pipeline state boundary: the renderer asks for state, not frames, so the server never becomes a frame renderer and hidden tabs can suspend render/audio.
- Suggested SPA/API stack: NOT RECOVERABLE FROM PLAN
- Two Postgres logical databases or schemas: the rationale is separation of `account` PII and `sim` state, with separate roles and no email in `sim-db`.
- Small job table or Postgres `LISTEN/NOTIFY`: the plan says it is "enough at v1 scale" and "no Kafka."
- Matter-of-fact mail templates: the rationale follows the system-copy rule for magic links, exports, and visits.
- UUIDv7 primary keys: NOT RECOVERABLE FROM PLAN
- Encrypted email plus keyed hash: ciphertext protects email at rest while the keyed hash supports lookup without using raw email as a key.
- Stable bird UUIDs: identity is "never recycled," names are string updates, and migrations must copy vectors in place.
- Append-only events with idempotency: events are consumed by the tick, and `unique (account_id, client_event_id)` prevents retries from double-counting presence.
- Snapshot DTO with quantized tiers: `sat_tier` and `vocal_tier` let the client render without reconstructing the vector; "Do not send the floats."
- Presence payload revalidation and rate limits: the tick re-validates the conjunction, and a lying client cannot invent more than one accepted ping per 20 seconds per session.
- Magic-link request always returning 202: NOT RECOVERABLE FROM PLAN
- Magic-link rate limits: the rollout instrumentation names the issue/consume ratio as "abuse," grounding rate limits per email hash and IP.
- Export JSON emailed as a download link: this avoids streaming personality vectors in-browser and accidentally rendering numbers in the UI.
- Event POST per-item results: NOT RECOVERABLE FROM PLAN
- Event future/past bounds: NOT RECOVERABLE FROM PLAN
- Visit client omits listen-in, offer, settle, interactive notebook, and the presence pinger: the rationale is read-only ambient visiting with no event writes.
- Visit log with no badge and no push: the rationale is that visit history should not become a reward, notification surface, or social graph.
- Revoked or expired visit token copy: the matter-of-fact sentence "This visit is no longer available" follows the system-copy rule.
- System copy on sign-in, expiry, timeout, and load failure: the plan says to use PRD sentences verbatim and forbids naturalist voice on these routes.

### 3. Simulation engine and sync model

- Catch-up backgrounding after the first 30 ticks: the rationale is that a long absence should not block first-bird time to first byte; current lighting and mood apply immediately, and SSE can push the rest.
- Drift as a non-negative leaky integrator: traits only accept non-negative increments, so attention can accumulate but neglect does not create negative personality drift.
- Drift calibration fixtures: one week should be "measurable in fixtures" but "invisible in motion"; three weeks should become "Visible in retrospect."
- Ambient-not-negative absence behavior: when presence is zero for many days, deltas stay zero; greeting probability may drop, but traits do not decrease and neglect never raises wariness via the vector.
- Settle in the drift model: settle contributes no drift terms, only ending presence and setting the settled flag.
- Mood transitions: mood responds to time of day, weather, recent events, contagion, and personality as "transition bias," not as a displayed stat.
- Mood persistence: mood is stored and not reset on snapshot, so a tab open does not snap birds to `content`.
- Return-greeting scoring and staggering: picking one primary greeter, maybe one secondary greeter, and never firing all birds at t=0 keeps greetings varied and non-mechanical.
- Weather and ambient events: rain and wind are fields on `aviaries`, not user-facing chrome; they affect call rate and mood without becoming dramatic weather events.
- Perch selection: hysteresis prevents pacing, and 4-10 second eased interpolation avoids teleporting.
- Call grammar runtime: a fixed species grammar preserves recognizability for a bird's life while mood changes timing and intensity.
- Snapshot `onset_plan`: the rationale is chorus alignment "without a second protocol."
- Bird-to-bird calls and wary contagion: these are "social-system signals, not user-facing stats."
- Adoption, naming, and identity: no starter catalog-pick, one allowed signup fly-in, age-gated "a new bird is near," and non-recycled IDs keep identity stable and adoption non-reward-like.
- Offer resolution partial tick: the rationale is that offer and settle reactions should arrive "in the next 1s, not the next minute," while remaining server-side.
- Single canonical record: there is no CRDT because laptop and phone both read snapshots and personality exists in one row.
- Device alignment: hidden tabs stop render, audio, and pings while the simulation continues, so devices pull the same canonical snapshot when visible.
- Preventing the lunch-overwrite bug: clients cannot patch personality; concurrent event streams add deltas in ingest order instead of last-write-wins.
- Visit vs host enforcement: a visit cookie has `visit:read` scope and the gateway forbids event writes, preventing visits from driving drift.
- Soft-deleted account restore: sign-in is allowed but the aviary is hidden behind a single system-voice restore screen until undelete or hard-delete.

### 4. Frontend rendering pipeline

- Scene composition and top bar: the scene is one horizontal aviary, chrome lives outside it, and subdued palette plus fading top bar keep UI from dominating the birds.
- Responsive scene sizing: the rationale is that "every bird stays on-screen"; narrow layouts compress spacing and wide layouts add air, but the aviary never crops birds.
- First frame and loading: no spinner, no ready pop, no logo sting, quiet-field HTML, and mid-cycle bird poses protect the "already running" feeling.
- Idle micro-motion: birds are "never paused-looking"; even drowsy birds retain tiny breathing-scale motion.
- Ambient leaves and feathers: they are "client-only ornaments" and "not in sim-db," keeping them out of canonical simulation state.
- Transitions: perch changes are eased arcs "not teleport"; day/night is a continuous palette mix "not a cut"; weather is "Never assertive."
- Reduced-motion mode: it is a designed renderer with still poses and crossfades, not `animation: none`, and shipping it after launch is "a v1 fail."
- Interaction targeting: NOT RECOVERABLE FROM PLAN
- Notebook UI: read-only panel prose, no edit/delete/annotate, and no visit-streak or user-behavior templates keep the notebook naturalist and non-gamified.

### 5. Audio pipeline

- Procedural WebAudio graph with no samples: motif tables stay small and the plan explicitly says "No sample files in the bundle."
- Seeded procedural calls: the same seed makes audio and captions match while still making calls unique across openings.
- Chorus mixing: "Real simultaneous voices, not stacked loops" and a low-ratio compressor let calls bloom instead of pumping.
- Listen-in mix: 1.6-2.2 second ramps avoid cuts, and non-focused birds are lowered but "never 0."
- WebAudio fallback: if audio fails, the aviary stays silent, captions turn on, and there is "No recorded-audio path. Unconditional."
- Autoplay handling: the visual aviary does not wait for audio, there is no "click to enable sound" toast, and blocked audio is treated as temporary fallback until a gesture.

### 6. Accessibility surfaces

- Screen-reader narration: the plan wants prose, not state dumps; it forbids "mood:", `boldness`, `perch 2`, and personality numbers in narration.
- Captions: captions come from the same seed and grammar as the sounding call, so text and audio stay aligned.
- Keyboard and focus: tab, arrows, Enter, Esc, offer, and settle paths make the aviary reachable without a pointer.
- Contrast and settings: WCAG AA applies to copy, captions, settings, and errors; settings use matter-of-fact voice as a named exception.
- Visit mode accessibility: visitors get the same narration, captions, and reduced motion, but no interactive bird actions beyond leaving the visit.

### 7. Performance, observability, rollout, testing, and voice

- CI performance budgets: JS size, first bird, idle FPS, memory, snapshot size, and tick p99 are gates so performance protects the first-frame and continuous-scene experience.
- Implementation tactics: inline critical CSS, defer modules, draw silhouettes before plumage detail, reuse audio nodes, virtualize the notebook, suspend hidden rendering, and code-split routes are all aimed at the stated budgets.
- Synthetic and RUM measurement: the plan measures load, first-bird paint, frame-time histograms, audio errors, and tick latency as operational categories only.
- Deliberately unmeasured analytics: per-bird interaction rates, offer funnels, heatmaps, drift distributions, visit frequency, and ML bird features are excluded because they could reconstruct "a relationship."
- Unsupported user agent handling: the response is matter-of-fact, with "No polyfill mountain."
- Build sequence: foundations precede sim, scene, audio, interactions, notebook, adoption, visits, account lifecycle, and perf CI so privacy, server authority, and core rendering exist before product ramp.
- Visits last among product features: the plan says visits wait until the read-only snapshot is cheap and event writes are locked down.
- Birds-per-aviary ramp: launch starts everyone with two birds, cap remains seven, and age gates are the only ramp; no staff override should email an "unlocked a bird" announcement.
- Feature flags: internal flags are allowed for audio and SSE, but reduced motion, captions, and narration cannot be flagged to a later train because they are launch blockers.
- Instrumentation from day one: tick p99, snapshot latency, first-bird mark, audio errors, event errors, magic-link abuse, and soft-delete counts are allowed; drift dashboards averaging traits are not.
- Launch bar: drift fixtures, presence conjunction tests, accessibility pass, performance jobs, and privacy CI define the release threshold.
- Testing strategy: unit, contract, presence, render, audio, accessibility, performance, and privacy tests mirror the plan's main risks and enforce the forbidden leaks.
- Copy module with `naturalist` and `system`: this codifies the voice split so aviary surfaces stay naturalist and account/error/settings surfaces stay matter-of-fact.
- New surfaces defaulting to `system`: money, identity, errors, and settings default to system voice so sensitive flows do not use the naturalist tone.
