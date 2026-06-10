## System-level intent

- **The server simulates, the client performs.** The plan names this as "the one sentence that governs every decision below." Canonical state "advances only on the server tick"; the client renders "an expressive, ephemeral performance layer" and "never writes canonical state directly." This shows up again in the "two-layer simulation boundary," the snapshot/event contract, the tick worker, the call scheduler, and the rule: "if losing it would matter, it is canonical; if losing it is invisible, it is performance."

- **One canonical continuously-advancing simulation, not divergent client simulations.** The plan repeatedly protects "the aviary continued without you," "multi-device sync," "no-last-write-wins," and "identity continuity." Hot ticking plus "exact lazy catch-up" preserves "bit-identical state to per-minute ticking" while keeping the system canonical.

- **Sync is made structural, not conventional.** "Single-writer architecture makes sync a property, not a feature." The plan uses leases, DB grants, idempotent event UUIDs, additive deltas, monotonic snapshots, and a separate simulation worker so "the phone-overwrites-laptop failure" is "unrepresentable."

- **Quiet private presence, never gamification.** The plan excludes "streaks, badges, levels, counters, visit calendars," "Tamagotchi mechanics," leaderboards, co-presence, and re-engagement. It says presence anti-abuse is "explicitly not built" because "there is no leaderboard, no streak, no economy - nothing to win."

- **No neglect penalty; drift is monotonic expression, not damage.** In the drift section, "absence simply contributes zero," there is "no decay term, no neglect penalty," and "quieter when ignored" is only an "expression effect." This keeps returning users' birds from having lost anything.

- **Privacy is enforced by shape, not policy.** The telemetry registry "rejects any metric with a per-account or per-bird dimension"; production accounts are not used for drift calibration; email exists on "exactly one table"; analytics has no simulation DB connection; exporting `interaction_events` to a pipeline is denied by DB role.

- **Observe the aviary, not the user.** Notebook inputs are "aviary observations only"; the generator input schema "cannot reference user-behavior aggregates." The copy linter flags "any string referencing user visit frequency anywhere."

- **Notice, never announce.** The voice system bans announcement-register strings, "welcome," "achievement," "streak," "level," second-person address, and announcement framing in naturalist surfaces. The plan says this is how "notice, never announce" survives contributors.

- **Naturalist register and matter-of-fact register are a hard boundary.** Notebook, narration, captions, and offer/adoption copy use the naturalist voice; "money/identity/errors/settings" use matter-of-fact copy. System surfaces must not borrow naturalist phrasing.

- **Accessibility ships with the feature.** Accessibility is "part of each feature's definition of done from M1" and "there is no accessibility milestone." Reduced-motion is "a distinct render mode" and "ships at launch."

- **Performance budgets are product constraints.** The plan treats first-bird render, bundle size, frame rate, memory growth, and tick latency as "CI-enforced gates" with "red = no merge." It says the "conceit dies" if first paint slips.

- **Recognizability is a structural guarantee.** Stable `birds.id`, persisted `traits`, fixed `call_seed`, and server-owned call parameters ensure a bird's identity and sound are preserved. Drift changes "how often and how elaborately Pip calls, never what Pip sounds like."

## Per-feature whys

### Scope

- **Web SPA, evergreen browsers only:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts and one canonical aviary per account:** The plan keeps one canonical aviary so account state has one simulation root and avoids shared or multiple aviaries, social-network surfaces, and co-presence complexity.

- **Magic-link email auth:** Magic links avoid passwords and SSO, and `POST /auth/magic-link` returns "202 always" so account enumeration is not exposed.

- **Per-device revocable sessions:** Sessions are per-device so users can see a device list and revoke a session without affecting the account itself.

- **Email change with verification:** The plan calls for "verify-before-commit email change," preserving identity/account integrity before the address changes.

- **Account export:** Export is a "data-portability escape hatch" because "the user's data is theirs." It uses an emailed signed link and includes otherwise hidden vectors in machine-flavored keys.

- **Soft-then-hard deletion:** `deletion_requested_at` suspends ticks and sign-in shows restore; the daily reaper hard-deletes after 30 days. This gives a restore window and then full cascade deletion.

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Age-gated growth to a cap of seven:** `next_adoption_offer_at` depends on `aviary.created_at` alone, so "there is no code path from engagement to birds." The GA rollout says the age-gated third-bird offer makes the population ramp slowly "by design."

- **Server-side simulation tick:** The tick owns personality drift, mood, weather, notebook entries, and canonical state so multi-device sync, identity continuity, and "the aviary continued without you" fall out of the architecture.

- **Single-screen aviary scene:** NOT RECOVERABLE FROM PLAN

- **Return-greeting:** The greeting must land within "1-2 seconds of tab-open" even though tick cadence is a minute, so the snapshot includes a server-computed greeting directive that the client performs.

- **Listen-in:** Listen-in foregrounds one bird while keeping the others at an ambient floor, and its start/end events feed drift inputs plus caption and narration priority hints.

- **Offers with per-bird cooldown:** Offers let accepted or nearby offerings nudge curiosity and boldness as small server-side drift inputs while remaining quiet interactions, not badges or economy. The exact seed / song fragment / still pool choice is not separately justified.

- **Settle with 5s undo:** Settle is "mood-quieting only" and ramps the aviary toward near-quiet; undo restores the previous ramp within the short window without creating drift.

- **Field notebook, read-only and sparse:** Notebook entries are an "observer's-record" and are generated once, stored as written, and kept sparse so active aviaries do not turn into a feed or expose template fatigue.

- **Presence accounting by three-signal conjunction:** NOT RECOVERABLE FROM PLAN

- **Procedural call synthesis:** Procedural WebAudio avoids a recorded-audio path and supports personal, recognizable calls whose signature anchors come from species and `call_seed`.

- **Chorus mixing:** Independent synth voices make overlap "true polyphony," avoiding "phase-cancel artifacts" and honoring the plan's reason for refusing loops.

- **Graceful-silence fallback with captions default-on:** Browser autoplay and WebAudio failures can block audio, so visual calling animations plus captions carry the moment without an announcement banner.

- **Visits through per-invite email links:** Visitors do not need accounts because sign-up "would convert a quiet affordance into a funnel."

- **Read-only ambient visitor view:** Visitor endpoints have a separate narrower serializer and no event endpoint, so co-presence and visitor interactivity are "unrepresentable."

- **Revocable 30-day visit invites:** Invites expire and can be revoked, and visitor polling bounds revocation latency to one poll interval.

- **Silent visit log:** Visitor presence writes nothing to the host event log; `visit_sessions.last_poll_at` exists only to compute visit-log duration, keeping visits separate from interaction state.

- **Default-off visit notification toggle:** Email is transactional only; there is "no other email path," which keeps visit notification from becoming a re-engagement surface.

- **Accessibility surfaces at launch:** The plan says a post-launch reduced-motion mode would be "a launch that excluded those users," so narration, captions, keyboard, contrast, and reduced-motion are launch requirements.

- **Performance budgets:** The budgets protect the product feel and catch regressions at merge: first bird under 500ms, 60fps idle, bundle cap, and zero memory growth.

- **Aggregate-only operational telemetry:** RUM is "operational health only"; the metric schema has no account/bird dimension so leaderboard-shaped and population-interaction analysis features are architecturally unreachable.

- **Out-of-scope gamification, Tamagotchi mechanics, social surfaces, push, payments, shared aviaries, user-customizable scenes, SSO/passwords:** These are "enforced, not just deferred" so the product stays quiet, private, and non-game-like.

### Adjudicated ambiguities and interpretation decisions

- **Export includes personality vectors:** The never-expose rule governs product surfaces, while export is data portability. Obfuscation would be "dishonest in the wrong direction."

- **Hot ticking plus exact lazy catch-up:** Per-minute ticking for idle aviaries is wasteful; pure deterministic catch-up preserves the continuous canonical simulation while keeping compute "proportional to attention."

- **Client performance layer:** Minute snapshots cannot create second-to-second aliveness, so the client can perform idle motion, call scheduling, and greeting choreography as long as nothing persists upstream as canonical state.

- **Server-directed, client-performed return greeting:** Canonical boldness, mood, and absence length choose the greeter and class; client variation lets the moment land quickly without waiting for a tick.

- **Canonical account timezone:** Multiple devices and visitors make "the user's local time" ambiguous, so one IANA timezone follows the most recently present device; visitors see the host clock and travel re-anchors over about 30 minutes.

- **Synthetic cohorts and consented beta calibration:** Production interaction behavior cannot be used because of the privacy commitment, so drift calibration uses deterministic simulated users and explicit-consent beta accounts only.

- **Temporary first-session silence:** Autoplay restrictions may block audio before a gesture, so the plan uses the same graceful-silence register as WebAudio fallback and fades in on the first input.

- **HTTPS polling instead of WebSockets:** The product model is pull-based, revocation may wait until the next snapshot, and polling matches tick cadence while simplifying edge caching and connection-state bugs.

- **No presence anti-abuse:** Spoofing presence has no reward; anti-cheat would spend privacy and complexity "to protect nothing."

- **Canvas 2D renderer:** Seven birds and ambient effects fit Canvas 2D, with a smaller bundle than WebGL and no WebGL context-loss handling. WebGL is only an escalation path if profiling fails.

- **Visitors do not need accounts:** Requiring sign-up would turn a friend's birds into a funnel instead of a quiet affordance.

### Architecture

- **API service in Node.js/TypeScript:** TypeScript end-to-end keeps the call-grammar parameter model, prose/voice engine, and deterministic state-shaping code in one implementation shared by server and client.

- **Simulation worker:** It is the sole writer of canonical bird/aviary state, with sharding and leases so exactly one worker owns an aviary at a time.

- **Email sender:** The queue sends magic links, invite links, export links, and opt-in visit notifications only; because no other email pipeline exists, re-engagement email is not available to abuse.

- **Two-layer simulation boundary:** The boundary preserves canonical cross-device state while allowing expressive local performance. The rule is "if losing it would matter, it is canonical; if losing it is invisible, it is performance."

- **Append-only interaction events:** Clients submit observations, not values. UUID idempotency and server receipt order make retries safe and canonical ordering explicit.

- **State snapshots with `tick_index`:** Snapshots carry monotonic versions so clients can discard stale reads and never render backward.

### Data model

- **Email stored encrypted on exactly one table:** This protects identity data and lets linting/code review ensure no other table, log line, queue key, or metric carries email.

- **Stable `birds.id`:** The row id is the bird identity; migrations, renames, and species-pool changes must preserve it so identity continuity is never reset.

- **Stored `traits`:** Traits are state, not runtime derivations from the event log. This avoids serving from replay and keeps each bird's accumulated personality intact.

- **Fixed `call_seed`:** The seed anchors call-signature recognizability across mood and drift.

- **Stored notebook prose:** Entries are generated once and persisted so a template change cannot silently rewrite history, preserving the observer's-record framing.

- **Deletion cascade:** Because aggregate telemetry has no account dimension, hard deletion only needs to cascade canonical stores and export/email residue; there is "nothing to scrub" in aggregate telemetry by construction.

### API surface

- **Auth endpoints:** `POST /auth/magic-link` returns 202 always to avoid account enumeration, and the consume endpoint is single-use with a 15-minute expiry.

- **Snapshot pull triggers:** Visibility change, long render-frame gap, keepalive, settle, and adoption all pull snapshots so resumed or changed scenes converge on canonical state.

- **Event batching and `sendBeacon`:** Events batch while active and flush on hidden so "presence end must not be lost to tab close."

- **Visitor router:** The visitor endpoint set is separate, read-only, has no notebook, no greeting directive, and no event acceptance, making visitor interactivity impossible rather than disabled.

- **Rate-limit values:** NOT RECOVERABLE FROM PLAN

### Simulation engine design

- **Pure tick function:** Determinism and counter-mode PRNG make ticks reproducible for tests and incident forensics, and make fold catch-up equivalent to sequential ticks.

- **Transactional per-tick pipeline:** State writes and event consumption are committed together so a lost lease or retry can re-run exactly.

- **Drift function with bounded non-negative deltas:** Daily budgets and monotonic deltas enforce "a single session never moves a personality value visibly" and ensure absence contributes zero rather than decay.

- **Expression-threshold calibration tests:** Synthetic cohorts assert week-1 instrument drift and day 17-28 expression thresholds, turning the three-week promise into an assertable property.

- **Mood model:** Mood decays toward time-of-day baseline instead of hard-resetting, so tab-open and overnight catch-up do not snap state.

- **Weather state machine:** Weather is canonical so visitors and devices see the same rain, and weather can feed mood modifiers consistently.

- **Server-owned call-grammar parameters:** The server owns signature, density, tempo, ornamentation, and chorus hints, making each bird personal without server-side audio synthesis.

- **Notebook noteworthiness scoring:** Inputs are aviary observations only, enforcing the line between observing the aviary and observing the user.

- **Notebook sparsity governor:** The token bucket targets about one entry per 2-4 days, so high activity does not produce a feed and low-grade observations are dropped.

- **Adoption pacing by aviary age:** Age-only adoption keeps bird growth separate from engagement, and decline leaves the offer open without nagging.

### Sync model

- **One writer invariant:** DB roles prevent the API service from updating canonical-state columns, so a bug fails at the database rather than silently violating architecture.

- **Additive deltas in event order:** Clients never send trait values, so phone-overwrites-laptop cannot happen.

- **Monotonic snapshots:** Devices can differ briefly by one tick but converge at the next poll because neither wrote state.

- **Lease correctness:** At-most-one worker ownership plus atomic state/event transactions makes a lost-lease re-run exact.

- **Offline integrity check:** Nightly replay detects "silent drift-loss bugs" before they become a user's vague unease.

- **No aviary-state conflict UI:** The plan does not create conflict UI because "no aviary-state conflict can exist."

### Frontend rendering pipeline

- **Preact/React DOM shell plus Canvas 2D scene:** The DOM handles top bar, notebook, settings, and dialogs; the canvas scene stays outside the framework loop for frame performance and bundle control.

- **Layered offscreen canvases:** Separating sky, foliage, birds, foreground, effects, and DOM overlays lets dirty tracking keep idle cost low.

- **Bird animation system:** Mood-specific behavior trees, personality-scaled motion, smoothed noise, and canonical `perch_zone` make birds feel alive without storing animation as state.

- **Snapshot interpolation:** Smooth transitions prevent birds from teleporting when canonical perch or mood changes arrive.

- **Greeting choreography variation:** The client uses timing and pose jitter within the server directive so no two greetings render identically.

- **First-paint path:** Inline critical CSS, a small bootstrap module, silhouette-first drawing, and progressive hydration protect the sub-500ms first-bird budget and avoid a spinner.

- **Route-level code splitting and delayed audio worklet:** Notebook, settings, invite, export, and adoption are lazy chunks, and audio is outside the first-paint critical path.

- **Reduced-motion mode:** It is art-directed as the same aviary through still-pose sequences and cross-fades, not a disabled-animation afterthought.

- **Lifecycle handling:** Hidden tabs stop rAF, suspend audio, and flush events; visible resumes pull a snapshot and render forward without a reconnecting state.

### Audio pipeline

- **WebAudio AudioWorklet synthesis:** The worklet keeps the render thread clean; pooled nodes and buffers serve the no-memory-growth rule.

- **Per-bird voice:** Species timbre plus `call_seed` gives each bird a voice identity while staying procedural.

- **Call scheduler:** Read-only signature invariants mean motif skeleton, pitch center, and timbre cannot be tuned away by accident.

- **Chorus:** Independent synth instances provide true polyphony, and a gentle master compressor prevents stacked choruses from becoming harsh.

- **Listen-in mix:** A focused bird comes forward while others remain at an ambient floor "never -infinity," keeping the aviary present during focus.

- **User mute:** Muted sessions still count presence fully; mute only mildly damps vocal-frequency drift and is "never a penalty."

- **Audio fallback:** Graceful silence with captions and visual calling carries the birds' vocal life; there is no recorded-audio path.

### Accessibility surfaces

- **Screen-reader narration:** Narration reads the same canonical snapshot and local events as the visual scene, in "field-notebook voice," avoiding state lists, trait labels, and mood labels.

- **Captions:** Captions are generated from actual scheduled call parameters, so they "can never describe a call that didn't happen."

- **Keyboard:** DOM focus proxies over canvas birds, keyboard shortcuts, dialogs, and settle undo make scene interaction reachable without pointer input.

- **Contrast:** Automated checks cover every lighting state, not only the default, so AA contrast holds across morning, midday, evening, night, and settled palettes.

- **Accessibility testing:** Axe, manual screen-reader scripts, narration snapshot tests, and reduced-motion screenshot suites prevent late accessibility regression.

### The voice system

- **Shared prose engine:** A single isomorphic engine supplies notebook, narration, captions, and offer/adoption copy so the naturalist voice is consistent.

- **Copy linter:** Register declarations and CI bans make "notice, never announce" survive contributors and block streak/visit-frequency/user-behavior language.

- **Register boundary:** The PRD rule is encoded as matter-of-fact for money/identity/errors/settings and naturalist for everything else.

### Performance budgets and observability

- **Budgets as CI gates:** Bundle size, first bird, frame rate, memory growth, and tick latency are merge gates because they define whether the product feel survives.

- **Synthetic monitoring:** Scripted browsers from several geographies measure load, first-bird render, frame timing, and audio-context errors as operational health checks.

- **Aggregate-only RUM:** Metrics cover timings, errors, latencies, and histograms without account or bird dimensions, keeping observability operational rather than behavioral.

- **Infrastructure boundary:** Analytics cannot connect to the simulation database, and the simulation database has no analytics replica, preventing interaction-event leakage by architecture.

### Security and privacy engineering

- **Hashed tokens, encrypted email, scoped visitor tokens, CSRF, CSP, rate limits, deletion, and export links:** The plan groups these as commitments already woven in, protecting account identity, invite access, session integrity, and data portability.

### Rollout

- **M0 prototype spikes:** Procedural calls and Canvas idle-motion are de-risked before dependence; exit criteria are listening-panel approval and frame-time numbers, not demos.

- **M1 canonical core:** Schema, auth, event ingest, worker leases, drift/mood/weather, snapshots, and calibration harness land with the nightly replay-integrity job as definition of done.

- **M2 the aviary:** Renderer, behavior, greeting, listen-in, offers, settle, presence, first-paint, adoption, accessibility, reduced-motion, voice engine, and copy linter are built together so the core experience is not separated from access or voice.

- **M3 notebook, accounts, visits:** These surfaces are grouped after the aviary core, with notebook generation, matter-of-fact account surfaces, invite flow, visit log, and notification toggle.

- **M4 hardening and beta:** Consent-based beta verifies drift bands, expression checks, high-bird-count audio, notebook sparsity, performance, accessibility, and privacy red-team work without production behavior mining.

- **GA adoption ladder:** Everyone starts with two birds, and the age-gated third-bird offer makes population bird-count ramp slowly by design, so no separate feature-flag ramp is needed.

- **Kill-switches and degradations:** Audio, weather, notebook generation, invites, and per-aviary ticks can degrade quietly; there is no banner system and no kill-switch that turns into an announcement surface.

### Tuning-constant register

- **Single server-side config home:** Drift weights, budgets, presence windows, mood weights, weather, greeting thresholds, cooldowns, ramps, notebook rates, adoption ladder, and keepalive interval are changeable without deploy and logged as values, not per-user data.
