## System-level intent

1. **A quiet relationship surface, not a game loop.** The plan frames Pocket Aviary as "a browser-only, single-aviary relationship surface" and repeatedly excludes "Gamification," "Tamagotchi mechanics," "streaks, scores, badges, hunger, death," and "public social surfaces." This intent also appears in the PR checklist questions: "Does this announce instead of notice?", "Does this punish absence?", and "Does this expose a number the user optimizes?"

2. **Absence must never be punished.** The plan makes this a non-negotiable: "Drift is monotonic toward expressive; neglect never moves traits down." The drift section restates it as the "Monotonic expressive rule" and explains that neglected birds can become "quieter/less greeting-forward without becoming less bold/colorful/warm," implementing "ambient, not mistrust."

3. **Canonical bird identity lives on the server.** The plan says "Server owns all canonical simulation state; clients never write personality or mood," and later calls tick the "sole writer of vectors, moods, weather, scheduled intents." This is tied to the highest-priority risk narrative: reset vectors, swapped bird IDs, or merge bugs are "silent and irreversible emotionally" and cause "identity loss."

4. **Presence means real, recent attention, not an open tab.** The plan defines presence as "the conjunction of visibility + focus + recent input activity - not 'tab open.'" It fixes an activity window that is "long enough for still watching; short enough to stall overnight laptop" and names "Population drifts overnight" as a risk if presence definition leaks.

5. **Expressivity should emerge over weeks.** The product sentence says birds "drift slowly toward expressive over weeks." Drift rates are calibrated so change is "measurable; not session-obvious," with three-week dogfood before "marketing anything about personality." The risk table says drift too fast becomes a "Tamagotchi numbers game" and drift too slow becomes a "Screensaver."

6. **Aliveness should feel procedural and natural, not canned.** The plan requires "Procedural calls only; no recorded call loops, even as fallback." Call grammar preserves species motif identity through seeds and parametric atoms, and greeting variation avoids "unison fanfare" and toast because a canned greeting makes the "Product becomes theater."

7. **The product voice should notice rather than announce.** The plan bans "announcement UI," says there is "No toast" for greetings, uses "matter-of-fact error bodies," and tells PR reviewers to ask "Does this announce instead of notice?" Notebook prose is constrained to naturalist language with "no user streak language" and a banlist including "achievement," "streak," "XP," and "you visited."

8. **Sparsity is a product value.** The notebook target is "~1 entry / 3-5 active days" with "Sparsity over feed." Notebook generation is salience-gated so it does not become an "event log," and rollout instrumentation forbids engagement funnels that imply "streak thinking."

9. **Accessibility is affective, not just compliant.** The plan says "Accessibility is a designed surface shipping with v1, not a retrofit." Later, it names "A11y as labels-only" as a risk because it causes "Exclusion of affect"; narration design reviews should evaluate "charm, not only WCAG checklist." Reduced motion must be a "designed aesthetic" and "ship day one."

10. **Social contact must stay ambient, read-only, and opt-in.** The plan includes only "Opt-in email visit invites" and "read-only ambient visitor," with visitor presence explicitly not feeding host drift. "Visit feature scope-creep" is framed as "Social network gravity," mitigated by "Only listed endpoints; no presence from visitors."

11. **Privacy-safe infrastructure is part of the product identity.** The plan separates telemetry from simulation storage, forbids "personality values in logs" and "email in logs," uses account UUIDs rather than email, and requires a "privacy architecture test" that warehouse connectors cannot select simulation tables.

12. **Performance is part of the first impression of aliveness.** The budgets include "Time to first bird visible" under 500ms, 60fps idle, and no 30-minute heap growth. First paint must avoid T-pose and spinner states; tick backlog is a risk because the "Aviary feels frozen/outdated."

13. **Calibration beats pretending the feel is knowable up front.** The plan explicitly makes "defensible" ambiguity calls and labels them "calibration point." It includes CI simulation of synthetic weeks, "three week club" human calibration, and server-configurable knobs so rates can change "without redeploy."

## Per-feature whys

### 0. Purpose and posture / 1. Scope

- **Browser-only modern web SPA:** The plan ties this to the product sentence: a "browser-only, single-aviary relationship surface." It also keeps native iOS/Android apps out of scope for v1.

- **Email magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Per-device sessions and revoke sessions:** NOT RECOVERABLE FROM PLAN

- **Email change with verify:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** The why is a coherent "single-aviary relationship surface" and one shared "wallet of truth" across devices, rather than multi-aviary accounts or device-specific birds.

- **Two starter birds:** The starter pairing why is to use "fixed curated pairs" with "complementary traits," "avoid catalog," and "ensure audible contrast."

- **Age-gated adoption up to seven birds:** The plan says the third bird at day 45 "deepens without attention-farming," later birds "spread over ~1 year," and the cap is protected by server enforcement and listening tests.

- **Server tick around 60s:** The rationale is explicit: it "Matches '~once per minute'; simple scheduler math."

- **Presence triad:** Presence requires visibility, focus, and recent input so the system does not treat "tab open" as presence. The 180s window is "long enough for still watching; short enough to stall overnight laptop."

- **Presence ping interval:** The 30s ping while the triad is true is "Dense enough for tick consumption; light enough for mobile."

- **Personality vectors and drift:** The why is to let birds "drift slowly toward expressive over weeks" while keeping all deltas monotonic so absence does not move traits down.

- **Mood machine:** The mood enum includes `settled` because it "supports evening/settle/night"; mood transitions use phase, weather, offers, listen-in, neighbors, and personality so birds do not snap to a generic state.

- **Procedural calls:** The plan requires procedural calls because recorded call loops, including fallback loops, would violate the product posture. The risk table says "Audio uncanniness / loops" causes "Spell break."

- **Return-greeting:** The rationale is to let birds notice return through animation and call, with "No toast," "never unison fanfare," and a11y narration priority, avoiding a greeting that "feels canned."

- **Listen-in:** The plan's articulated why is to focus one bird without silencing the aviary: focused gain ramps up while others stay at an "ambient floor (never 0)."

- **Offers:** The offer tray creates seed, song fragment, and still pool interactions whose outcomes feed curiosity and boldness drift. The 4-minute cooldown "Prevents single-session curiosity saturation."

- **Settle:** The settle gesture biases birds toward drowsy/settled for the remaining session and "does not rewrite personality," preserving the no-punishment, no-trait-rewrite rule.

- **Field notebook:** The notebook is auto, sparse, and read-only because the product values "Sparsity over feed" and must avoid becoming an "event log" or using streak/gamification language.

- **Screen-reader narration:** The why is that accessibility is a "designed surface" and should carry affect; the risk table rejects "A11y as labels-only" because it excludes affect.

- **Call captions:** Captions support call meaning and accessibility; they are generated from the same call params and are forced on if audio is unavailable.

- **Multi-device sync:** The plan wants the laptop and phone to see the same `tick_version` "wallet of truth," with slight delay acceptable but no "choose this device's birds."

- **Opt-in email visit invites and read-only ambient visitor:** The why is to allow ambient visits without social-network gravity: default off, revocable, read-only, no visitor POST events, and no visitor presence contributing to host drift.

- **Export JSON snapshot:** Export includes personality numbers because it is "the user's data portability copy," while those numbers remain hidden in product UI.

- **Soft delete then hard delete:** The articulated why is privacy and trust: delete honors "privacy windows" and hard purges birds, events, notebook, and telemetry-linkable rows.

- **Performance budgets:** The budgets protect first-bird visibility, 60fps idle, no memory growth, and tick freshness so the aviary does not feel frozen, bloated, or delayed.

- **Aggregate RUM and privacy-partitioned telemetry:** The why is privacy-safe observability: aggregate route, timing, tick, and error data are allowed, while personality values, emails, and per-bird interaction streams are forbidden.

### 2. Architecture

- **Monorepo with deployable boundaries:** The plan says the monorepo should have "clear deployable boundaries" across web, API, simulation, mailer, notebook, narrator, infra, and shared packages.

- **API with Postgres, Redis, and object storage:** Postgres is the "system of record," Redis handles sessions, rate limits, and short-lived tokens, and object storage is for short-lived export artifacts.

- **Simulation worker pool with pure sim-core:** The plan puts pure logic in `sim-core` so ticks are "deterministic given seed + inputs" and unit-tested.

- **SPA client with no SSR of bird motion:** The why is first-bird performance: the plan prefers an HTML shell plus bootstrap snapshot for TTFB and says no SSR of bird motion is required.

- **Client writes interaction events while server appends the log:** This keeps account identity and canonical personality/mood on the server while still accepting presence, listen-in, offer, and settle events from the client.

- **Snapshot-to-client render pipeline:** The server snapshot is "truth"; the client store uses an immutable snapshot plus local clock offset, and visual/audio/a11y layers interpolate from that truth.

- **Pure client ornaments:** NOT RECOVERABLE FROM PLAN

- **Hard ban on client PATCH personality:** The reason is canonical identity: "no code path, admin tool, migration, or 'sync helper'" may let the client patch personality fields.

- **Separate telemetry sink:** The plan forbids analytics warehouse joins to the simulation DB so telemetry remains aggregate-only and privacy-partitioned.

- **Account UUID rather than email as primary key:** The rationale is privacy and identity stability: account IDs are "never email," while email is encrypted and used only for auth, invite, and export delivery.

- **Stable bird UUID through rename:** The why is bird identity continuity: a rename "does not change ID."

- **Derived render hints instead of raw trait names:** The plan avoids raw vector names in client code comments/UI and says no debug panel, because personality numbers must not be exposed in product surfaces.

### 3. Data model / 4. API surface

- **Append-only interaction events:** The event log supports ordered server processing, idempotent `client_event_id`, processed tick barriers, and no absolute personality/mood writes from clients.

- **Notebook source event provenance:** NOT RECOVERABLE FROM PLAN

- **Visit log with redacted visitor email:** The plan balances host transparency with privacy: keep the host-supplied email for transparency but store visitor email redacted and hashed.

- **Species pool as data modules:** Species ship as "data modules, not code forks," keeping species content separated from code paths.

- **Starter species not chosen by user:** The plan says the system picks complementary starters so the user does "not choose," avoiding a catalog and ensuring audible contrast.

- **Client-local transient types not durable:** NOT RECOVERABLE FROM PLAN

- **Export including personality numbers:** The rationale is data portability: numbers are included only because the export is the user's copy, while they are "still never shown in product UI."

- **Matter-of-fact errors and no welcome payloads:** This preserves the product voice: API errors should be matter-of-fact, while the bird return-greeting supplies the affect without "welcome" UI payloads.

- **Magic-link request returning 202 always:** The plan gives the why as "anti-enumeration," with rate limits per email and IP.

- **Snapshot `tick_version` and optional delta:** The why is freshness and shared canonical state; clients can use ETag/`tick_version` and poll or long-poll while visible.

- **Events API recomputing trust:** The server stores raw claims but "recomputes trust" to reject implausible presence, spam, and "backdated storms."

- **Duration fields preferred over absolute set state:** NOT RECOVERABLE FROM PLAN

- **No notebook write APIs:** The rationale is that the notebook is "auto, sparse, read-only" rather than a user-authored feed.

- **Visitor snapshot omitting owner controls:** The why is read-only ambient visiting: visitors get birds, weather, day, and calls but not notebook, offers, settle, adoption, or settings.

- **Keepalive, polling, and visibility freshness:** The plan uses snapshot polling, immediate pull on visibility/pageshow, and event flush on pagehide so the client stays fresh through suspend and bfcache.

### 5. Simulation engine design

- **Tick transaction steps:** The tick loads state, summarizes events, applies drift/mood/perch/call/notebook, and bumps `tick_version` so simulation changes are atomic and retry-safe.

- **Drift calibration tests:** The tests make sure regular presence is measurable over synthetic weeks, zero presence keeps personality flat, and spam offers cannot jump curiosity beyond a session cap.

- **Expressivity energy cache:** This exists so greetings and approach rates can decay after absence while stored personality never decays, implementing "ambient, not mistrust."

- **Mood persistence across sessions:** The plan says moods persist and advance toward phase-consistent attractors while the user is away, "not snap to content on open."

- **Perch and idle intent mapping:** NOT RECOVERABLE FROM PLAN

- **Call grammar motif identity:** The rationale is recognizability: species motif family identity stays fixed by `call_grammar_seed` and species while mood changes ornaments, not family.

- **Chorus and answer behavior:** High `social_warmth` birds answer neighbor calls within 400-1200ms, and the mix avoids overlap so chorus remains recognizable rather than muddy.

- **Weather generator:** NOT RECOVERABLE FROM PLAN

- **Return-greeting planner:** The planner varies greeting by absence, mood, boldness, energy, and staggered RNG so the result is not a toast, not a fanfare, and not canned theater.

- **Offer cooldown and outcomes:** Cooldown prevents "single-session curiosity saturation," while accepted/positive outcomes feed curiosity, boldness, and vocal behavior drift.

- **Notebook generation salience and rate limiter:** The job looks for notables and rate-limits entries so the notebook remains sparse naturalist prose instead of a product-wide event log.

- **Adoption offer in chrome:** The plan presents a "soft naturalist offer" rather than "toasty confetti," matching the no-announcement and no-attention-farming posture.

### 6. Sync model / 7. Frontend rendering pipeline

- **Canonical single writer and conflict prevention:** The tick is the sole writer of vectors and moods; personality uses additive server deltas rather than LWW or CRDT merges to avoid silent identity loss.

- **Offline and suspend resume:** On resume, the client pulls a snapshot, discards stale interpolation, and rebuilds motion from snapshot poses to preserve the sense that the aviary was "already running."

- **Matter-of-fact error surfaces:** Expired links, session timeout, snapshot failure, and visit unavailable pages should be direct and should "Never naturalist-snipe."

- **Scene graph layers and no pan/zoom/scroll:** NOT RECOVERABLE FROM PLAN

- **First paint with no spinner and no T-pose:** The plan wants bootstrap or parallel snapshot fetch and phase offsets so "nothing locks in T-pose"; quiet field appears only if the snapshot is not ready, with "no spinner."

- **Reduced motion crossfade backend:** The why is accessibility as design: reduced motion is a `MotionBackend = Full | Crossfade` and a "designed aesthetic," not a degraded afterthought.

- **Top bar fade and no badges:** The top bar fades into the scene and avoids badges or notification dots, matching the plan's no-announcement, no-gamification surface.

- **Canvas2D or light WebGL recommendation:** The plan prefers these because SVG-only "may struggle with 7 birds + filters at 60fps."

### 8. Audio pipeline / 9. Accessibility surfaces

- **WebAudio blocked fallback:** If WebAudio is unavailable, the feature becomes "silence + captions forced on" with "No MP3 pack," preserving the procedural-call rule and accessibility.

- **Listen-in gain ramp and ambient floor:** Listen-in focuses one bird while others remain audible at an ambient floor, so the scene does not collapse into solo isolation.

- **Mixing max simultaneous full voices:** The why is explicit: "Preserve recognizability over density."

- **Screen-reader live region priority:** Narration prioritizes return-greeting, offer reaction, settle, and ambient updates so screen-reader users receive the same naturalist affect rather than trait numbers or perch indices.

- **Captions toggle and auto-on audio failure:** Captions provide short call descriptions near the bird and are forced on when audio fails.

- **Keyboard navigation:** NOT RECOVERABLE FROM PLAN

- **Contrast and caption scrims:** The reason is WCAG AA: chrome text and captions must remain AA against day/night palettes and local scrims.

### 10. Performance / 11. Security / 12. Rollout / 13. Testing / 14. Risks / 15-20 Execution controls

- **Code-splitting, deferred notebook, no large audio samples, font subsetting:** These tactics exist to meet TTFB and bundle budgets while keeping first bird visible quickly.

- **Observability forbidden dimensions:** The plan forbids per-bird streams, personality values, emails, and account-ID session duration dimensions to keep telemetry privacy-safe and not product-analytic.

- **CI perf gates and a11y/tone tests:** The gates enforce bundle size, memory soak, sim drift calibration, axe checks, and narration banlists so core promises do not regress.

- **Magic-link and visit security controls:** Hashing tokens, single-use 15m magic links, revocation, rate limits, and unguessable visit tokens address enumeration, abuse, spam, and live invite revocation.

- **Privacy architecture test:** The why is to enforce the storage boundary: warehouse connectors must not be able to select simulation tables.

- **Soft launch with dogfood drift observation:** The plan requires dogfood accounts and at least three weeks of drift observation before "marketing anything about personality" so expressive timing is calibrated.

- **Birds-per-aviary ramp:** The launch and adoption ramp exists for "simpler chorus calibration" and to prove "audio recognizability" at higher bird counts.

- **Content freeze tone linter:** The linter blocks strings like "welcome back," "streak," "achievement," "hunger," "XP," "leaderboard," "feed the," "game over," and "daily goal" to protect product identity.

- **Human calibration loops:** The "three week club" slows clocks if drift is visible day-to-day and bumps `k_p` if invisible at day 21, keeping the feel in the intended weekly timescale.

- **Highest-priority identity-loss mitigation:** Backups of `personality_vectors`, rehearsed migrations, and not rebuilding vectors from replay as sole truth exist because identity loss is "silent and irreversible emotionally."

- **Execution-oriented work breakdown:** NOT RECOVERABLE FROM PLAN

- **"Implement next" order:** NOT RECOVERABLE FROM PLAN

- **Glossary lock:** The plan locks domain terms like `Bird`, `Aviary`, `Call`, `Mood`, `Drift`, `Presence`, `ListenIn`, `Offer`, `Settle`, `FieldNotebook`, and `Visit` and avoids `pet`, `xp`, `hunger`, and `solo` to keep code vocabulary aligned with product identity.

- **Closing PR checklist:** The checklist enforces the core philosophy on every change: no announcing instead of noticing, no punishing absence, no exposed optimizing numbers, no client personality writes, no recorded audio, no streak/social graph/discovery, no a11y hole, and no PII or per-bird romance in telemetry.
