## System-level intent

- **Build an ambient product, not a distressed one.** The plan states this directly in "Expressiveness gate (neglect without punishment)" and calls it the "engine-level implementation of 'ambient, not distressed.'" It recurs in "After two weeks away, they are quieter, not wounded" and "The user never owes the birds a visit."
- **Treat watching as the product.** Presence is tuned around the premise that "watching without moving is the product," and the scope says that if a ticket teaches the user that "presence is for a counter, it is rejected."
- **Keep the server as the source of truth.** The plan repeatedly says "server is the only writer," "Every host device is a projector," and "The browser never advances mood, perch intent, weather, or personality."
- **Allow immediate surface reactions while keeping personality drift slow.** The fast offer path exists because "Reactions must be immediate; drift must stay slow." Offers and greetings mutate "ephemeral" snapshot fields, while personality "still waits for the tick."
- **Reject game and announcement pressure.** The plan bans "scores, quests, streaks, badges, levels, XP," "milestone celebrations," "green-dot calendars," "visit badges," and "welcome toasts." It closes with "Build the quiet window, not a game around it."
- **Make privacy an architectural boundary.** The plan has a dedicated "Privacy as architecture" section: "No analytics role," "Email ciphertext lives on accounts.email_ciphertext and nowhere else," and "Every log line, span, queue name, cache key, and filename uses account_id UUID."
- **Use two deliberate voices.** "Product surfaces -> naturalist; system surfaces -> matter-of-fact." The plan also says shared prose should give "One voice across notebook, live region, captions," while auth failures must not use naturalist voice.
- **Ship accessibility as the same product.** "Accessibility ships on the same day as the visual aviary," and reduced motion is "a designed aesthetic," not a boolean `animate: none` or "v1.1."
- **Preserve the conceit that the aviary is already alive.** The boot sequence forbids "spinner," "logo splash," and "fade-from-black"; first paint is a "quiet field," first frame is "mid-action," and load-state leakage is a risk because it contradicts "already alive."
- **Protect bird identity over art or naming changes.** Bird ids are "STABLE forever," migrations "may never delete a bird row and insert a replacement," and "identity is the bird id, not the species."
- **Prefer restraint and cheap continuity.** The tick cadence aims for "Continuity without burning compute on abandoned accounts," the stack is chosen for "one language" and a "small team," and the team note says: "When torn between a richer feature and restraint, cut the feature."

## Per-feature whys

### 0. Decisions and scope

- **Presence activity window:** The 4-minute window exists because the plan says the PRD asks for "a few minutes, lean longer" and because "watching without moving is the product."
- **Presence ping cadence:** The 30s ping cadence is "Dense enough to survive a dropped packet; sparse enough to stay cheap."
- **Multi-device presence:** Presence uses "Union of time, never sum" to prevent "dual-device inflation of the dominant drift input."
- **Tick cadence:** The 60s / 5 min / lazy catch-up tiers provide "Continuity without burning compute on abandoned accounts."
- **Trait range:** `numeric(6,5)` in `[0.00000, 1.00000]` is "Small, stable, enough precision for week-1 instrument deltas."
- **Starter trait band:** New birds seed in `[0.22, 0.42]` to leave "Room to drift toward expressive" and so the two starters "read as different animals, not twins."
- **Mood enum:** The five moods are chosen because they "Match PRD examples"; night sleeping stays `drowsy` plus a render flag rather than adding a sixth mood.
- **Offer cooldown:** The 4-minute per-bird per-offer-kind cooldown "Stops curiosity/boldness saturation inside one session."
- **Notebook generation:** The deterministic compiler and phrase banks avoid live LLM and warehouse-side generation for "Voice control, privacy boundary, reproducibility."
- **Renderer:** Canvas 2D plus DOM chrome is chosen for the "60fps budget," "mid-action first frame," and so "captions/focus rings stay in DOM."
- **Event log:** Postgres append-only with UUID keys avoids "PII-in-partition-key failure"; v1 volume "does not need a log cluster."
- **Account timezone:** The IANA timezone follows the user because "Day/night and mood must follow the user, not UTC."
- **Song library:** Six built-in fragments are enough for a "small library" while staying "inside bundle" and "no recorded audio."
- **Visit opt-in notify:** Visit notification is email only, off by default, because the PRD allows a settings toggle and "email is the only v1 channel."
- **New-bird pacing:** Bird 3+ pacing is based on age, "not attention," for "Months-scale deepening."
- **Greeting absence bands:** Different bands exist so a "coffee-return" feels different from a "two-day return."
- **Session snapshot auth:** Cookie session plus bearer on fetch lets "the HTML document and XHR share one session."
- **Fast offer path:** Offers and greetings return synchronously because "Reactions must be immediate"; personality remains on the tick because "drift must stay slow."
- **Narration author:** A shared server-side prose package gives "One voice across notebook, live region, captions."
- **Export personality numbers:** Traits are allowed only in JSON takeout because the PRD "explicitly includes vectors in export and forbids showing them."
- **Stack:** TypeScript, Next.js, Node.js, Postgres, Redis, and edge delivery are chosen for "One language, last-two-browser support, small team."
- **No client sim:** The browser does not advance canonical state because this "Prevents dual-device fork."
- **Browser-only Pocket Aviary:** NOT RECOVERABLE FROM PLAN
- **Out-of-v1 gamification, Tamagotchi mechanics, and social network surfaces:** The plan rejects them because "presence is for a counter" is a failure mode, absence must not be "failure," and the product must not become "a game around it."

### 2. Architecture

- **Edge/web, API, event log, sim worker, Postgres shape:** The diagram and process split make "service boundaries" explicit and keep the tick worker the "only writer of personality, mood, perch intent, weather, notebook candidates."
- **Redis duties:** Redis holds short-lived auth, rate-limit, revoke, and export tokens, but is "never a source of personality."
- **`apps/web`:** The web app owns "Render, input, WebAudio, presence signals, interpolation" and must not own personality or grants so the browser stays a projector.
- **`services/api`:** The API owns "HTTP, auth, validation, snapshot reads, event append" and not "Drift math," preserving the sim boundary.
- **`services/sim`:** The sim owns "Tick, drift, mood, perch intent, weather, greeting plan, offer reaction, notebook compiler" and not telemetry warehouse, keeping product state out of analytics.
- **`services/mail`:** Mail excludes bird fields from templates so mail remains transactional and not an aviary ping.
- **`packages/prose`:** Prose owns "Naturalist + matter-of-fact string tables and compilers" while forbidding "Numbers for UI," preserving the voice split.
- **Privacy split:** Separate `sim` and `ops` schemas prevent analytics from reading birds, vectors, events, notebook, or snapshots.
- **Email ciphertext and HMAC lookup:** Email is encrypted in one place, lookup uses `email_hash`, and logs/keys/files use UUIDs so email does not become an identifier.
- **RUM and synthetics without bird details:** Metrics carry "no bird ids, no species, no interaction types beyond HTTP route" to avoid product-funnel instrumentation.
- **Repo layout:** NOT RECOVERABLE FROM PLAN

### 3. Data model

- **UUIDv7 primary keys:** They are "time-ordered, not email-derived."
- **Lazy account and aviary creation on magic-link consume:** This "avoids account spam from typed-wrong addresses."
- **Per-device revocable sessions:** NOT RECOVERABLE FROM PLAN
- **Stable bird identity:** Bird rows may not be deleted and replaced because ids are "STABLE forever"; species-pool edits "remap visuals/motifs in place."
- **Expressiveness gate:** The gate implements "neglect without punishment"; a bold bird away for two weeks becomes "quieter, not warier" and snaps back "by being watched - not by apology."
- **Event log:** Client events are append-only and idempotent, and "Clients never send trait values" because drift belongs to ordered server processing.
- **Snapshot row:** Snapshots are "materialized, not derived from full history," matching the architecture's "sim reader (hot row)" role.
- **Snapshot exclusions:** Raw traits, visit counts, streak-like fields, "days since last visit," and other accounts are absent to avoid personality-number UI and counter-like surfaces.
- **Notebook sparsity:** The notebook is "read-only, sparse" and token-bucketed so entries remain observations rather than a session log.
- **Visitor snapshot stripping:** Visitors get the host snapshot minus greeting, reactions, offers, notebook, and listen-in so visits are read-only and birds do not appear to notice the visitor.
- **Indexes without ranking aggregates:** The plan forbids indexes or views that rank aviaries by visits, bird count, or age because there is no public surface and no "just in case" aggregate.

### 4. API surface

- **Uniform magic-link response:** `/v1/auth/magic-link` always returns the same 202 body to avoid revealing account existence.
- **Magic-link consume:** One-time consume and replay-as-error preserve single-use authentication; unknown accounts are created only during consume.
- **Email change verify-first:** The old email remains until consume so the new address is verified before account email changes.
- **`POST /v1/aviary/events`:** Idempotence on `client_event_id` supports safe batched event ingest.
- **`POST /v1/aviary/apply`:** The fast path writes an event and returns an updated snapshot but "Does not write personality," preserving immediate reaction plus slow drift.
- **Event ingest timestamp windows:** NOT RECOVERABLE FROM PLAN
- **Presence ingest requiring three true signals:** Partial pings are ignored because the server does not "upgrade" a partial ping.
- **Snapshot pull reasons:** `start`, `visible`, and `wakeup` may attach greetings; `poll` is "Cheap read" and "No greeting."
- **Visit flow:** The visitor route uses the same renderer but removes top-bar actions and greeting playback so "Birds do not re-greet," the host snapshot is not mutated, and the host is "not told in-session."
- **Revocation:** Next visitor snapshot returns 410 and there is "No host toast," matching the quiet/no-announcement principle.
- **Error voice:** Errors are matter-of-fact, with "No bird names" and no "welcome back," keeping system surfaces out of naturalist voice.

### 5. Simulation engine

- **Tick worker transaction:** One aviary tick is a single transaction so event folding, drift, mood, weather, perches, notebook, snapshots, and processed markers stay coherent.
- **Long-idle catch-up:** Simulated 5-minute steps preserve deterministic time effects while capping request work and still serving "the latest completed snapshot."
- **Presence reconstruction:** Gaps close windows, overlapping devices merge intervals, settle/session_end closes presence, and credit is host-only, protecting honest presence and union-not-sum drift.
- **Drift function:** Traits move "only upward" and daily caps discard unused cap to prevent "hoarding"; settle gives "zero" trait delta.
- **Calibration fixtures:** The fixtures are "test fixtures, not product analytics," including regular-week, overnight-tab, and neglect cases.
- **Mood transitions:** Mood is "a small Markov step each tick, scored - not random-walk cosmetics," persists across sessions, and never snaps to content on start.
- **Perch intent:** Unique zones/slots prevent overlap; `next_perch.eta` lets the client interpolate because "Teleporting is a bug"; the user cannot place birds.
- **Weather:** Weather is mild, never thunder/snow/modal, and RNG is seeded by `aviary_id + date` so catch-up is deterministic.
- **Call grammar runtime:** The server schedules windows and motif plans while the client plays the given plan, preventing client-invented call behavior.
- **Chorus handling:** Detune and slight delay prevent replies from phase-locking.
- **Return-greeting:** Greetings are based on absence bands and bird scores so short return, long return, and second-device openings behave differently without toast or "you've been gone."
- **Offer reactions:** The fast path validates the receiver, enforces cooldown, writes ephemeral reaction, may persist mood, and "Do not touch traits here."
- **Starter adoption:** The first two birds use different silhouettes and motif libraries and complementary traits so they are "different animals, not twins."
- **Age-gated adoption:** New birds appear from aviary age, not attention, and the surface is "naturalist, not a reward chest" with "No confetti."
- **Notebook compiler:** Inputs are "facts, never user-behavior tallies"; forbidden facts include session length, streaks, click counts, trait deltas, "you," exclamation, and achievement framing.

### 6. Sync model

- **Canonicality:** "One aviary row, one snapshot row, N birds" makes every host device "a projector" with no CRDT, no LWW document, and no authoritative client personality cache.
- **Client caching:** Clients may cache assets and a last snapshot only to "paint a quiet field faster," never as a write base.
- **Conflict prevention:** Personality, mood, perch, weather, expressiveness, and notebook each have a single writer or append rule, avoiding a merge UI.
- **Two-device offers:** Cooldown turns the second offer into `ignore`, so no user-visible conflict modal is needed.
- **Clock and sleep:** Server time plus wakeup refetch avoids local fast-forward after a sleep gap because that "reads as a skip-cut."
- **Soft-delete:** Tick still runs during soft-delete "so restore is coherent," while other events are rejected.
- **Hard-delete:** Wiping all sim/account data plus "No cold analytics copy exists to forget" makes deletion meaningful.
- **Export job:** Export JSON is the "sole place traits are serialized to the user."

### 7. Frontend rendering pipeline

- **Boot quiet field:** First paint uses the quiet field, not a spinner or splash, because the central conceit is that the aviary is already present.
- **First bird budget:** Time-to-first-bird is measured as "first canvas pixel of a bird silhouette" and drives critical JS, snapshot size, font, and code-splitting choices.
- **Empty aviary soft fly-in:** Empty aviary happens once after account creation; later errors stay on last-good or quiet field so the aviary is never presented as empty again.
- **Scene layout:** The world has no pan, zoom, or scene scroll, and scaling keeps all occupied perches plus an empty slot on-screen so a bird is never cropped.
- **Palette and settle:** The palette is "calm naturalist," day/night is continuous, and settle eases into evening regardless of clock.
- **Ambient ornaments:** NOT RECOVERABLE FROM PLAN
- **Idle micro-motion:** Loops are never paused while visible, use `motion_phase` for "First frame mid-action," and are seeded so two birds "do not clone."
- **Transitions:** Perch changes use paths and wing-settle, while reduced motion cross-fades; this preserves movement without teleporting.
- **Top bar:** Icons-only chrome fades down after inactivity, focus restores opacity, and there are no badges or unread dots.
- **Reduced-motion renderer:** Reduced motion replaces loops with still poses and cross-fades but keeps greeting, audio, captions, and slower sun shifts because it is "a designed aesthetic."
- **Frame budget:** DOM captions and focus rings keep screen readers and contrast "real," and performance degradation may drop ornaments or rain density but "Never drop birds."
- **Memory:** Fixed pools, virtualized notebook, no unbounded listeners, and a 30-minute soak enforce "no client memory growth."

### 8. Audio pipeline

- **Procedural calls:** Motif graphs and grains replace PCM so calls stay inside the bundle, avoid recorded fallback, and support recognizable bird identity.
- **Recognizability test:** Same-bird identification must be at least 80%; if it fails, the plan says "we do not raise the cap; we retune motifs."
- **Listen-in:** The mix is "place, not a DAW"; focused birds rise while others remain at a floor, "never 0."
- **Chorus mix:** Avoiding identical motifs at the same sample, with detune and delay, protects against phasey chorus.
- **WebAudio fallback:** If audio cannot run, the product stays silent and captions are forced on; there is "No MP3/OGG sprites" and "No silent-fail without captions."
- **Allocation rules:** Fixed voice and buffer pools plus no allocation in `process()` preserve the frame/memory budget.
- **Song-fragment offer:** The six fragments are motif plans rather than files, preserving the no-recorded-audio and bundle constraints.

### 9. Accessibility surfaces

- **Same-day accessibility:** Accessibility is not deferred: "No 'v1.1 for reduced motion.'"
- **Screen-reader narration:** A single polite live region uses server narration; it avoids every micro-shift because narration is "weather-to-climate of the scene."
- **Narration voice:** Lowercase, present-tense, specific narration avoids trait numbers and "mood: content," matching naturalist voice.
- **Keyboard path:** Tab order, arrow navigation, Enter/Space listen-in, Escape behavior, focus rings, and keyboard dialogs provide the "full keyboard path."
- **Captions:** Caption text comes from the same `MotifPlan.caption` as audio, anchors to the bird bbox, meets contrast, and never covers the top bar.
- **Contrast and chrome:** All user copy is WCAG AA; scene itself has no copy.
- **A11y refusals:** The plan refuses personality vectors as ARIA stats, static screenshot mode as the accessible product, high-frequency live-region spam, and naturalist voice on auth failures.

### 10. Performance budgets and observability

- **Budgets:** Bundle, first-bird, idle FPS, heap, snapshot size, tick p99, and mail budgets are gates, not aspirations.
- **Aggregate-only measurement:** Measurement is limited to navigation, route, frame, audio-error, tick, backlog, catch-up, and anonymous session-duration aggregates.
- **Deliberately unmeasured behavior:** Per-bird offers, listen-in, presence minutes, species popularity, visit frequency, and population boldness are excluded because those would become engagement or calibration dashboards.
- **Greeting-to-offer funnel:** The plan says not to measure it because "That would optimize announcement."
- **Synthetics:** Fixture aviaries test first bird, FPS, and audio without using real users; the synthetic account is excluded from future analysis.
- **Browser support:** Feature detection fails closed to an unsupported page; "No polyfill soup" protects the 2MB cap.

### 11. Frontend module map

- **Module map:** The section says it exists "so the team can staff it."
- **Presence monitor:** The monitor is "load-bearing" because it owns the three-signal conjunction, 4-minute window, 30s pings, `session_end`, and hidden-tab stop behavior.

### 12. Security, privacy ops, mail

- **Magic links, session tokens, invite tokens, and export tokens:** Hashing, single-use rules, TTLs, revocation, and rate limits implement the security/privacy ops boundary.
- **Matter-of-fact email templates:** Mail subjects avoid lines like "your birds miss you," preserving the no-guilt and system-voice stance.
- **No marketing list, third-party analytics SDK, session replay, or heatmap:** These exclusions preserve privacy and avoid engagement instrumentation.
- **CSP:** `default-src self`, limited `connect-src`, and no arbitrary script protect the web surface.
- **Visit log decryption:** Visitor email is shown to the host only and decrypted in-process, limiting where email appears.

### 13. Testing strategy

- **Engine tests:** Engine tests are marked "highest leverage" because drift, presence, mood, greetings, cooldown, monotonic traits, notebook copy, and determinism carry the core product integrity.
- **Notebook forbidden-phrase linter:** The linter enforces the ban on "you," "achievement," "streak," "visited every," and "vocal frequency."
- **Tick determinism test:** Same events and same clock must produce the same snapshot hash, preserving reproducibility.
- **Client tests:** First-frame, no-toast/no-welcome, keyboard, reduced-motion, listen-in floor, and presence tests protect the boot conceit, accessibility, and anti-announcement rules.
- **Sync tests:** Two-device interleaving, trait schema contracts, and laptop sleep tests preserve the single personality trajectory and no-teleport behavior.
- **Perf and a11y tests:** Bundle, heap, axe, live-region, and contrast tests enforce the gates called out earlier.

### 14. Rollout

- **Build sequence:** The sequence makes the "boot conceit" true before engine work, puts voice review behind a launch gate, ships reduced motion as a product, and leaves visits until "after host loop feels right."
- **Age-gate adopt rollout:** Bird 3+ logic can ship disabled because "first users won't see bird 3 for 56 days."
- **Internal staff aviaries:** Staff aviaries use "real presence, not synthetics only" before closed rollout.
- **Closed-list calibration:** The 4-minute activity window may be extended if watching-without-moving drops presence too often, but it "may not shorten below 3."
- **Public v1 visits:** Visits are available but "unprompted," with "no onboarding share step."
- **Bird cap ramp:** The cap stays 7 and must not be a "growth lever"; if audio ABX fails at 5, lower the cap.
- **Day-one instrumentation:** Ship synthetics, RUM aggregates, tick alarm, audio errors, first-bird mark, heap soak, and magic-link failure rate, but not engagement dashboards or population trait charts.

### 15. Risks

- **Drift calibration mitigations:** Fixtures, hardcoded presence conjunction, union-not-sum, daily trait cap, expressiveness gate, and no production dashboard guard against "Tamagotchi numbers" and "screensaver."
- **Sync correctness mitigations:** Rejecting trait keys, restricting DB credentials, contract tests, interleaving tests, and the review question "who writes this column?" address invisible stale-device failures.
- **Audio uncanniness mitigations:** No recorded fallback, variation, detune/delay, ABX tests, listen-in floors, and captions over sprites protect the user's sense that chorus is alive.
- **Accessibility regressions mitigations:** Reduced-motion renderer and narration are milestones, not post-launch, because a late "label the canvas" patch would create a "different, worse product."
- **Load-state leakage mitigations:** Quiet field, bootstrap snapshots, 2MB and 500ms gates, and starter species in the critical bundle protect "already alive."
- **Announcement creep mitigations:** Copy linter, no badge component, demand-only visit log, and a PR checkbox stop "toasts, visit badges, 'you've been gone 4 days.'"
- **Privacy leakage mitigations:** HMAC email hash, UUIDs, schema grants, analytics CI bans, and no third-party replay stop convenience from leaking privacy.
- **Tick cost mitigations:** Activity-tiered cadence, lazy catch-up, deterministic functions, lease workers, and alarms avoid a naive "every aviary every 60s" approach.
- **Offer fast-path race mitigations:** Same-aviary locking plus no trait writes in apply prevent a fast reaction from being overwritten by a stale tick.
- **Naming and identity mitigations:** Immutable `birds.id`, species-table art updates, and rename as `UPDATE name` prevent art or naming work from creating a replacement bird.

### 16. Team notes

- **Vocabulary:** "bird, call, listen-in, offer, settle, notebook, visit, tick, presence" is normative, while solo/select/chirp/pet/character are banned in surfaced code names.
- **Two voices:** Product surfaces use naturalist voice; system surfaces use matter-of-fact voice.
- **Absence:** "The user never owes the birds a visit. Absence is quiet, not failure."
- **Restraint:** "When torn between a richer feature and restraint, cut the feature."
- **Final product framing:** The team should "Build the quiet window, not a game around it."
