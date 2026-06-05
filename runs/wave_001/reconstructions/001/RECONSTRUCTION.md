## System-level intent

- **"Feels alive" means continuity, not activity theater.** The plan repeats this through the server-authoritative tick, hidden vectors that "drift over weeks," idle motion that is "never paused," the first frame loading with "motion already in progress," and the risk that a spinner would make the aviary appear to be "loading rather than continuing."

- **"Notice, never announce" is a product rule.** It shows up in the refusal of notifications, "welcome back" surfaces, streaks, and visit-frequency messages; in the field notebook's sparse cadence; in the quiet new-bird offer; and in the statement that "the bird greeting is the entire welcome."

- **"Charm from specificity" comes from small, rule-shaped details.** The plan locates charm in per-bird call grammars, personality-shaped timing and pitch, mood-shaped idle micro-motion, captions such as "a soft three-note rise," and prose that is "lowercase, present-tense, specific."

- **"Restraint over richness" governs scope.** The non-goals are described as "refusals," not postponed features. The plan refuses gamification, Tamagotchi mechanics, social-network surfaces, customization, recorded audio, payments, native apps, and user-extensible species because each would compromise the coherent, quiet product shape.

- **The voice is split by surface.** The plan states "naturalist for the product surface, matter-of-fact for system surfaces." Aviary, notebook, narration, offer prompts, and return-greeting narration use naturalist voice; sign-in, account settings, sync errors, accessibility settings, unsupported-browser, and WebAudio-unavailable surfaces use matter-of-fact voice.

- **The server is the "only writer of personality state."** This is called the "cardinal rule" and appears in architecture, data model, API, simulation, sync, risks, and engineering notes. It is the principle behind multi-device sync, no last-write-wins merges, no local personality cache, and no client-side drift.

- **Privacy is enforced by boundaries, not policy.** The synthetic UUID rule, encrypted email, aggregate-only telemetry, IAM/network separation, and the rule that the telemetry pipeline "never touches the per-account sim DB" all carry the intent that per-bird state and a user's relationship with the aviary must not leak into analytics.

- **Accessibility is part of the designed aviary.** The plan says accessibility is "first-class" and "built in from day one," with reduced motion as "its own designed surface," screen-reader narration in the same naturalist voice, generated captions, keyboard navigation, contrast, and a launch-gate audit.

- **Performance is affective, not just technical.** The bundle budget, 500ms "time-to-first-bird," 60fps idle motion, no 30-minute memory growth, procedural assets, CDN-edged snapshots, and the note that "the 500ms time-to-first-bird is the affective bridge" all make speed part of the product feeling alive.

- **Calibration is favored over hard-coded certainty.** Presence window, tick cadence, mood enumeration, call intervals, listen-in ramp, notebook sparsity, palette curve, and bird-offer thresholds are named as "tuning knobs, not architectural decisions," governed by harnesses, synthetic fleet signals, and manual listening.

## Per-feature whys

### Scope, refusals, and ambiguity calls

- **Single-user accounts; one aviary per account:** The plan's shape is "one user, one aviary." This supports a per-account simulation store and avoids shared aviaries, household profiles, and social surfaces that would change the product into something broader.

- **Magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Two starter birds and a cap of seven:** The plan gives a small starting flock and a hard cap so the aviary remains a legible small system. Later birds are tied to "the rhythm of a relationship deepening," not a score or paid tier.

- **Fixed small species pool:** The pool is fixed because a future "import a species" feature would compromise the "coherent-set feel." The notes reiterate "about six species" and no rarity tiers or paid species.

- **Server-authoritative simulation tick:** The tick makes the aviary continue beyond a viewer's current client and makes multi-device sync a property of the same record. It also prevents client divergence and last-write-wins personality loss.

- **Personality vector drift over weeks:** Drift is the long relationship mechanic: "measurable drift in instruments after ~1 week" and "visible drift to the user after ~3 weeks." It is monotonic toward expressive so absence is not punished.

- **Mood state per bird:** Mood gives the renderer, audio, and social behavior a small shared state: idle motion, calls, time-of-day modifiers, and visitor rendering all use the same mood snapshot.

- **Procedural call synthesis, per-bird grammars, and chorus mixing:** Audio is the "affective spine." Procedural synthesis supports the bundle budget, avoids canned audio, keeps calls varied, and lets a bird remain recognizable by ear across drift.

- **Idle motion:** Idle motion must be "mood-shaped" and "never reads as paused" because the aviary should feel alive even when the user is only watching.

- **Return-greeting:** The greeting replaces any textual welcome. It is a bird action chosen by "personality x absence-length," preserving "notice, never announce."

- **Listen-in:** Listen-in is meant to feel like "leaning in to listen, not like switching channels." The mix rebalances rather than muting the other birds.

- **Offer (seed / song fragment / still pool):** The plan grounds offers as interaction events that feed drift: accepted offers can push curiosity, offers near a bird can push boldness, and user-initiated offer events can also drive narration.

- **Settle:** Settle cleanly closes a presence window, quiets calls, and shifts lighting toward evening. The undo window keeps the gesture reversible in the moment.

- **Field notebook:** The notebook is the quiet noticing surface: naturalist voice, server-authored observations, sparse generation, and no templates that observe visit frequency or streaks.

- **Presence accounting:** Presence requires visible tab, focus, and recent pointer-or-key activity so the server is accumulating a real watching window. The initially long activity window is chosen because "watching birds without moving is the actual product."

- **Multi-device sync:** Sync is not treated as a separate feature because both devices pull the same server-owned canonical state. The plan's desired shape is simply that "the aviary is the aviary the first device is showing."

- **Optional visit invitations:** The invitation is the single social affordance, "read-only ambient," revocable, opt-in, and default off so the product does not grow profiles, feeds, chat, comments, or discovery.

- **Accessibility surfaces as a set:** Screen-reader narration, reduced motion, captions, contrast, and keyboard navigation exist so users with different access needs still experience an aviary that "feels alive," not a stripped variant.

- **Performance budgets:** The budgets force procedural audio, procedural visuals, code-splitting, bounded workers, and CDN-edged initial state so the first bird appears quickly and idle motion stays alive over long sessions.

- **Account export:** NOT RECOVERABLE FROM PLAN

- **Soft-delete and hard-delete:** NOT RECOVERABLE FROM PLAN

- **Aggregate operational telemetry only:** Telemetry is limited to counters and timings so nothing can reconstruct "a user's relationship with their aviary." The per-account sim DB is walled off.

- **Web-only / no native app:** The plan refuses native because the "data model and protocols are not designed around native-client constraints."

- **No gamification or streak surfaces:** The plan treats the "cumulative-effect argument" as final. Streaks, badges, levels, scores, calendars, XP, and "harmless" celebrations would teach engagement behavior the product refuses.

- **No Tamagotchi mechanics:** Birds do not die, get hungry, decay, or punish absence because the "absence is fine" promise depends on monotonic drift toward expressive.

- **No social-network surfaces:** Profiles, follows, feeds, public discovery, comments, leaderboards, and rankings are refused so visit invitations remain the only social affordance.

- **No shared aviaries / multi-aviary accounts / household profiles:** The rationale is the same "one user, one aviary" product shape; expanding household or multi-aviary ownership would change the per-account simulation premise.

- **No payments / paid tiers / in-app purchases:** New birds are gated by aviary age, "not visit count, not interaction score," because the plan refuses to teach that more attention or money earns more birds.

- **No notifications about the aviary:** The plan says "the aviary is the welcome." Push, email, banners, and welcome-back surfaces would violate "notice, never announce."

- **No customizable scenes or perch controls:** Perch position is "a signal the user reads, not a layout the user controls." Customization would turn simulation meaning into decoration.

- **No recorded audio:** The bundle budget and chorus mechanic require procedural synthesis; fallback is "silence + captions," not canned audio.

- **English-only naming language:** NOT RECOVERABLE FROM PLAN

- **Browser timezone for time-of-day:** The plan binds day/night to the browser timezone for the session because following a traveling user across timezones would make time-of-day mood signals incoherent.

- **Presence activity window initially four minutes:** The window is leaned long because the product is watching birds, and watching can involve not moving the pointer or keyboard.

- **Simulation tick cadence initially sixty seconds:** A slow cadence is enough for user-perceived behavior, and it can stretch to 90-120s if server load warrants.

- **Mood-state enumeration cap:** The final set is capped to keep transition tables bounded.

### Architecture, data model, and API surface

- **Five service boundaries with the per-account sim DB read by exactly one service:** The boundary keeps the canonical aviary state protected while allowing account, aviary, visit, simulation, and telemetry responsibilities to be independently deployed and reasoned about.

- **Edge gateway synthetic-UUID routing:** The gateway is the only place that resolves session tokens; below it, logs and routing use UUIDs so email never becomes a routing key or log identifier.

- **Account service:** The account service owns auth, magic-link issuance, session tokens, export, and deletion so system/account concerns stay out of the aviary simulation path.

- **Aviary service read path and event append:** The aviary service serves snapshots and appends interaction events but "does not compute drift," preserving the simulation engine as the sole writer of personality.

- **Simulation engine as separate worker pool:** The engine is isolated because tick, drift, mood, notebook, and narration have their own SLOs and must scale independently while remaining the only writer of personality state.

- **Visit service:** The visit service handles invites, revocation, and visitor pull while sharing the aviary read path and avoiding separate visitor state.

- **Telemetry sidecar and IAM separation:** Aggregate telemetry reads only operational counters; IAM and network boundaries enforce that analytics never touches per-account simulation state.

- **Client/server split cardinal rule:** The client sends events, never personality values, mood states, or perch positions. This makes multi-device sync and "no last-write-wins" fall out of one decision.

- **Render hot path, cold path, and reduced-motion path:** The split protects frame performance, moves state/network/UI updates off the per-frame path, and gives reduced motion a separate designed register.

- **Build order:** The order follows dependencies because retrofits to the append-only event log and synthetic-UUID rule are expensive.

- **No localStorage, IndexedDB shadow, or offline mode:** Offline is refused because it would require defining offline drift semantics, which the plan says the PRD deliberately refuses to do.

- **Redacted personality snapshot:** Numeric personality vectors are never exposed to the client; the client receives renderable state and maybe bands for narration, while "the numbers live on the server."

- **Append-only event log with DB role enforcement:** Append-only roles make the log the source of truth, prevent aviary-service read-back, and restrict debugging read access to a separate audit role.

- **Notebook entries server-authored and read-only:** Server authorship keeps notebook prose in the naturalist voice and prevents user/client edits from becoming another interaction or analytics surface.

- **Visit log host shape:** NOT RECOVERABLE FROM PLAN

- **Matter-of-fact API and sync errors:** Errors are system surfaces, so they use matter-of-fact voice rather than naturalist prose.

- **Small state pull, CDN caching, and visibility/suspend pulls:** The state snapshot is kept kilobyte-sized and cached for 5-10s to support the 500ms first-bird budget and recover cleanly after tab visibility or suspend gaps.

- **Event submission batching and idempotency keys:** Batching limits write traffic, while idempotency prevents double-writes across reconnects.

- **Server-sent narration stream:** The optional stream gives screen-reader narration a slow, polite update channel while keeping the same prose embedded in snapshots for polling clients.

### Simulation, sync, rendering, audio, accessibility, and rollout

- **Active-account tick with a rolling tail:** The engine ticks recently active accounts, including a rolling tail, to keep the aviary "alive" even with no recent events.

- **Low-pass additive drift function:** Drift uses additive deltas toward an expressive target, never absolute value writes, so re-running the same log is deterministic and absence does not cause negative drift.

- **Drift calibration harness:** The harness enforces the named behavior: measurable after about a week, visible after about three weeks, and no negative delta after thirty days of zero presence.

- **Mood transition table:** A small Markov-like matrix weighted by personality lets boldness, vocal frequency, time of day, ambient events, and recent interactions shape mood without unbounded state.

- **Bird-to-bird interaction:** Chorus joining, softly spreading wary states, and night behavior make the aviary read as "a small social system rather than a row of independent NPCs."

- **Notebook sparsity and pressure counter:** Entries appear roughly every few days, more often on noteworthy state changes, and never every session so the notebook notices without announcing.

- **Closed notebook and narration template systems:** Rule-based template + slot generation makes the voice auditable and keeps prose in the same product surface without LLM or freeform generation.

- **Narration string:** Narration lets a screen-reader user and a sighted user read "the same aviary"; "the prose is the only difference."

- **Snapshot delivery and conflict prevention:** Edged initial snapshots, visibility pulls, keepalives, append-only events, idempotency, and server-owned personality prevent stale or conflicting client state from becoming divergent aviaries.

- **Sync error surface:** Rare sync-adjacent failures use matter-of-fact strings kept in the system-voice table.

- **Single horizontal scene:** The renderer has no panning, scrolling, or zooming because "all birds are always in frame."

- **Sky and day/night palette:** Palette interpolation is continuous and local-time-bound so "evening arrive[s] over a few minutes" and never snaps.

- **Perch zones and heights:** Birds are placed by engine-owned perch state; combined with the refusal of drag-to-place controls, perch position remains something the user reads.

- **Focus indicator:** A soft high-contrast outline is required so focused birds remain visible against bright and dim palette states.

- **Top bar fade:** The top bar becomes nearly transparent after cursor stillness and returns on activity so system chrome stays quiet over the aviary.

- **Idle micro-motion vocabulary:** Preening, scanning, watchful posture, drowsy feather fluffing, and weight resets give mood-specific visible life while avoiding a paused read.

- **Flight transitions:** Moving between perches is an animated path rather than a teleport, so snapshot changes preserve continuity.

- **First frame and quiet loading field:** The first frame starts from current positions and current motions; a quiet field replaces a spinner so the aviary appears to be continuing, not waking up.

- **Reduced-motion cross-fade sequencer:** Reduced motion is a calmer, slower aviary with pose cross-fades, no parallax or leaf drift, and the same state changes; it is "not a broken-looking one."

- **Procedural and small bird visual assets:** Procedural silhouettes, palettes, saturation detail, small SVGs, and compact bitmaps are driven by the <2MB bundle budget.

- **Call grammar variety, pitch, timing, and mood shaping:** Motif variety prevents repeated canned calls; personality and mood keep each bird recognizable while making calls respond to drift and state.

- **WebAudio synthesis with bounded workers:** Short generated buffers, bounded worker pools, disposal after each call, and leak tests serve the "no memory growth over 30 minutes" rule.

- **Chorus mixer and listen-in decay:** Per-bird gain, spatialization, and gradual ramps create a scene mix where listening in feels like leaning closer and other birds "never go silent."

- **WebAudio fallback:** If audio is unavailable, the aviary runs in "graceful silence with captions on by default" because the no-recorded-audio rule is unconditional.

- **Generated call captions:** Captions are derived from motif, mood, and pitch, placed near the bird, and written in naturalist voice for audio-off, hearing-difference, and noisy-environment use.

- **Screen-reader narration with aria-live polite:** The narration cadence and voice match the field notebook so moving between surfaces still feels like one product.

- **Keyboard navigation:** Tab, arrows, Enter, Escape, top-bar shortcuts, and keyboard-navigable offers ensure all interactive surfaces are reachable without a pointer.

- **Contrast:** WCAG AA applies to user copy, especially chrome and captions, so text remains readable over the aviary scene.

- **Accessibility audit:** The audit is part of the v1 launch gate because accessibility is not a v1.x task.

- **Last two major browser versions:** The plan supports current Chrome, Safari, Firefox, and Edge and refuses very old compatibility paths because the cost-benefit does not justify bundle bloat.

- **What to measure and what not to measure:** Operational timings, errors, latencies, and anonymized histograms are measured; per-bird state and per-account histories are not, preserving the privacy boundary.

- **Synthetic perf fleet:** Automated browsers from common geographies are the primary signal for performance regressions between releases.

- **Bird-count ramp and new-bird offers:** New birds become available by aviary age, with quiet in-aviary offers, because the pacing should match "a relationship deepening" and not teach that attention earns stuff.

- **Day-one instrumentation:** Aggregate telemetry, synthetic perf fleet, drift harness, memory-leak test, audio uncanniness review, and accessibility audit all start at launch so the core promises are checked from v1.

- **Post-launch drift calibration loop:** Drift weights can be re-tuned from aggregated, anonymized deltas in the first 4-8 weeks while the harness protects user-visible behavior.

- **Manual audio calibration:** Pitch range, inter-call interval, and listen-in ramp are tuned by listening rather than telemetry because audio feeling is central and not reducible to counters.
