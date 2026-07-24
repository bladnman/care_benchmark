## System-level intent

- Calm, non-gamified attachment over scoring. This shows up in the non-goals banning "streaks, scores, badges, levels," "death, hunger, distress," and "Counters of 'birds adopted.'" The explicit UX rules say "Idle watching is interaction," "Neglect -> quieter expressivity, never distress art," and success means "multi-week 'knowing' a bird by ear and perch habits without stats."

- Server canonical truth over client-owned state. The plan repeatedly uses "server canonical state," "Simulation truth," "Only tick writes personality," and "Clients only append events." It rejects "client-owned personality state" and "last-write-wins personality merge," and names the desired outcome as a "multi-device morning/night same mood continuum."

- Quiet living scene before interface. The client is a "single-screen horizontal scene" with "no in-scene buttons," top-bar chrome that fades, "first pixel is living scene or quiet field," "no spinner," and no "Welcome back" surface. The product is meant to feel present rather than announcey.

- Expressive aliveness without punishment. Drift is "monotonic up toward expressive only," and "neglect" changes only transient greeting/front-perch expression, "leaving vector untouched." Risks call out "Tamagotchi feel" and "screensaver" as opposite failure modes.

- Naturalist product voice, matter-of-fact system voice. Notebook prose is "naturalist lowercase," screen-reader output is a "naturalist paragraph," and system surfaces use "matter-of-fact English." The plan explicitly says "Voice split enforced in copy review checklist."

- Accessibility as a first release surface, not a later patch. Scope calls "Accessibility as first-class surfaces," risk mitigation says "Ship narration + RMO + captions in same release," and success says an "A11y user receives living prose, not meter reading."

- Privacy-minimal operations. The plan allows "Aggregate operational telemetry only" and says "no per-bird analytics warehouse." It also says "simulation DB never ETL'd to warehouse," "no account/bird dimensions," and uses encrypted email plus internal UUIDs.

- Small v1 architecture with hard budgets. The plan rejects "microservice explosion," says "one deployable API + one worker pool is enough for v1," prefers a lightweight scene graph to hit "<2MB gzip JS," and treats tick, memory, first-bird, and bundle budgets as design constraints.

## Per-feature whys

### Scope and defensible calls

- Browser-only SPA: NOT RECOVERABLE FROM PLAN

- Single account and single canonical aviary with multi-device sync: Why: the plan wants one "server canonical state" so devices read the same snapshot stream and avoid "multi-aviary / shared aviaries" and personality merge ambiguity.

- Magic-link email auth: NOT RECOVERABLE FROM PLAN

- Two starter birds at account creation: NOT RECOVERABLE FROM PLAN

- Adopt up to seven birds, paced by aviary age rather than engagement metrics: Why: adoption must be "never pay-to-unlock," avoid a "rarity economy," and keep early months mostly "2-3" birds while audio/listen tests protect the cap.

- Server-side simulation tick: Why: the tick owns "personality drift, mood transitions, ambient weather, call-timing state" and lets the simulation continue as server truth even when clients are hidden or absent.

- Listen-in, offer, settle, field notebook, and presence accounting as client surfaces: Why: they are the sanctioned ways for a user to be with the aviary while preserving the rule that clients append events rather than own personality state.

- Optional quiet visits: Why: visits are "invite-by-email, read-only, off by default" so the feature stays quiet and does not become "profiles, follows, discovery, leaderboards, chat, comments, co-presence" or other social-network surfaces.

- Accessibility surfaces: Why: the plan says accessibility is "first-class," avoids an "ARIA dump of coords/mood enums," and wants "living prose, not meter reading."

- Aggregate operational telemetry only: Why: telemetry is for load, latency, long tasks, audio errors, tick health, and event lag, while "per-bird interaction funnels," "engagement streaks," and "cross-account 'average boldness'" are deliberately excluded for privacy and product-principle reasons.

- Tick cadence of 60s nominal, 45-90s under load, idempotent ordered processing: Why: the plan allows load flexibility but requires the tick to be "idempotent" and to "order events by server receipt time" so event processing remains safe.

- Three-minute presence activity window: Why: the plan says to "calibrate later, prefer long side," and later rollout validates "presence honesty vs laptop-asleep false positive."

- Trait range and starter sampling: NOT RECOVERABLE FROM PLAN

- Mood enum values: NOT RECOVERABLE FROM PLAN

- Four-minute offer cooldown: NOT RECOVERABLE FROM PLAN

- Notebook sparsity target: Why: notebook entries should be rare enough to stay like naturalist observations, with "no user-behavior scoring," "no numbers of traits," and mitigations against "generic spam."

- New-bird pacing by aviary age: Why: the third bird arrives at "presence-capable age" and later gates widen so adoption does not become engagement pressure or "pay-to-unlock."

- Species pool size and stable internal codes: NOT RECOVERABLE FROM PLAN

- One nocturnally active species: Why: night behavior can stay alive because evening/night rate reduction has an "except nightjar species" path.

### Architecture

- One deployable API plus one worker pool: Why: "No microservice explosion" is the stated rationale; this shape is enough for v1 if tick cost is bounded and presence work is small.

- Simulation workers: Why: workers are the canonical place to pull due accounts, consume unread events, write bird/aviary rows, and emit notebook candidates without letting clients write the simulation truth.

- Transactional mailer: Why: email exists for "magic links, export links, visit invites" only, matching the non-goal of no push/email about the aviary except magic-link and optional visit-notification toggle.

- Static SPA delivery through CDN and chunked routes: Why: this supports the performance plan for CDN static assets, small initial JS, and code-split settings/visits/export.

- Server-owned personality vector: Why: the plan prohibits visible or client-owned personality numbers and names the risk of "silent personality loss" from dual-device merge bugs.

- Server-owned mood, perch intent, weather, and settle flag: Why: these are part of "Simulation truth" so sessions and devices stay on the same mood/perch/weather continuum.

- Client-append interaction and presence events: Why: client activity becomes durable input to the server tick without "overwriting with absolute client values."

- Client-only visual interpolation, leaf drift, and audio buffers: Why: presentation can stay smooth and procedural while never inventing personality.

- Snapshot seeds for client call phases: Why: the client may synthesize calls "offline between snapshots" while still deriving from server seeds.

- Hidden-tab behavior: Why: stopping rAF and suspending WebAudio protects resources, presence ends honestly on hide, and the simulation still "continues on server."

### Data model and API surface

- Account UUID as the only internal key, with encrypted email and keyed email hash: Why: account routes should key by session to account UUID, "never from client-supplied email," while the hash supports lookup and rate-limit.

- Sessions with device labels and revocation: NOT RECOVERABLE FROM PLAN

- Bird stable lifelong identity: Why: a stable bird id supports lifelong recognizability, including pitch offset "locked for life of bird id" and habits by ear/perch.

- Append-only event log with server_received_at, processed_at, actor, and session: Why: events must be ordered by server receipt, processed idempotently, separated by host/visitor actor, and consumed by the tick instead of merged client-side.

- Notebook entry prose plus hidden trigger: Why: the visible surface is "naturalist lowercase" prose; the internal reason code is "not shown" so the notebook does not expose scoring machinery.

- Visit invites and visit sessions: Why: visit access is tokenized, expiring, revocable, read-only, and measurable only as host visit log, preserving "quiet visits."

- MagicLink token hash, expiry, and consumed_at: Why: magic links have "15 min" expiry and "single consume" replay protection.

- Event, bird, and version indexes: Why: they support unprocessed event reads ordered for the worker and cache/ETag snapshot versioning.

- Auth errors and account-system copy in matter-of-fact English: Why: system surfaces should avoid product theater and follow the plan's matter-of-fact voice rule.

- Export, delete, and undelete routes: Why: account ops support privacy, soft-delete, and a 30-day undelete window, with risk mitigation for cascading hard delete and email expunge.

- Snapshot payload with render fields but no personality names: Why: the UI must not expose "personality numbers"; only anonymous presentation fields like call rate, approach bias, and plumage may leak when needed for rendering.

- Snapshot stream with version bumps rather than per-frame updates: Why: the server publishes canonical changes at tick/settle cadence while the client handles per-frame presentation.

- Batched POST of aviary events with 202 and rate validation: Why: clients append interaction facts while the server validates rates and later processes them in tick order.

- Adopt route where the server picks species: Why: species selection stays server-controlled, age-gated, and outside any rarity economy.

- Visit snapshot route without host drift writes: Why: visitor events "never affect host drift," keeping visits observational.

### Simulation engine design

- Lease-based tick loop: Why: claiming accounts with a lease and ordered loads lets workers be idempotent, avoid duplicate mutation, and keep tick p99 observable.

- Time-of-day lighting phase from locale_tz: Why: day cycle and mood time-of-day should match the account locale and support a morning/night continuum.

- Rare Markov weather: Why: weather is ambient, short-lived, "few rains/week," and acts through temporary mood biases rather than permanent trait changes.

- Host presence aggregation: Why: "Idle watching is interaction" for the host, but only host presence should drive drift.

- Interaction deltas rather than absolute client values: Why: drift is additive from the ordered log and avoids last-write-wins personality merge.

- Mood transitions with personality, time-of-day, weather, offers, and listen-ins: Why: moods persist across sessions, advance offline, and respond first to recent session events.

- Perch choice from boldness and mood with anti-collapse behavior: Why: the aviary should show habits without all birds "collapsing front unless extreme."

- Bird-to-bird chorus and wary contagion: Why: overlapping high-vocal windows create a chorus flag and short-lived contagion, adding group aliveness without permanent penalties.

- Notebook candidate heuristics during tick: Why: prose fires only on rarity heuristics and quota, keeping notebook entries sparse.

- Drift as low-pass, monotonic up toward expressive only: Why: focused presence should make birds more expressive over weeks, while neglect never reduces traits or creates distress.

- Drift calibration targets: Why: too fast creates "Tamagotchi feel"; too slow becomes "screensaver"; single-session deltas must stay below user-perceptible thresholds.

- Trait-specific weights for presence, listen-in, and offers: Why: different actions gently shape social_warmth, vocal_frequency, curiosity, and boldness without direct user-visible stats.

- Ambient quietness on neglect: Why: absence changes greeting and front-perch priors only as a transient mood/intent modifier, preserving the vector and avoiding punishment.

- Procedural call grammar with server seeds and client WebAudio: Why: calls should be recognizable yet "never identical twice" and avoid recorded-audio call libraries.

- Chorus and listen-in ducking: Why: listen-in focuses a target bird while leaving other birds at an "ambient floor > 0" so the scene remains alive.

- Return greetings ranked by boldness, social_warmth, and not-wary state: Why: greetings should reflect bird personalities, absence buckets, and staggered seeds, not "three canned clips."

- Adoption species rules and rename: Why: starters are complementary, later species are random unused-or-repeat, there is "no rarity economy," and rename is free.

- Rule-based notebook generation: Why: templates can produce lowercase naturalist observations in v1 without LLM generic spam, session-log feel, trait numbers, or behavior scoring.

### Sync model

- Canonical single writer: Why: personality and authoritative mood/perch/weather/version have one writer so there is "no CRDT personality merge."

- Initial boot, visibility, sleep-resume, keepalive, and SSE refresh triggers: Why: clients should rejoin canonical state after gaps and reconcile optimistic settle/offer UI on the next version.

- Conflict prevention for personality, offers, settle, magic-link replay, and soft delete: Why: ordered logs prevent LWW, server cooldown truth wins, simultaneous settle events are both valid, links consume once, and deleted accounts answer with matter-of-fact 410.

- Visit isolation: Why: visitor snapshots exclude host session IDs and visitor events never enter host drift aggregation.

### Frontend rendering pipeline

- TypeScript SPA with lightweight custom scene graph preferred: Why: the bundle must stay under the initial JS gzip budget and avoid heavy game-engine cost.

- Layered single scene with no in-scene buttons: Why: the aviary should remain a living field rather than chrome, with sky, foliage, perches, birds, and occasional foreground leaf.

- Responsive perch layout that never crops birds: Why: phone layouts should compress gaps while preserving visible birds.

- Quiet field placeholder instead of spinner: Why: the plan explicitly says first frame should be a "quiet field" and "never branded spinner."

- Mid-animation bird placement from motion seed and server time: Why: first render should feel already alive, not reset to a static or empty pose.

- Return greeting within 1-2s when ready: Why: greetings are part of the return experience, but audio waits for readiness and autoplay constraints.

- Empty aviary only once at first adoption fly-in: Why: the product should not show a dead empty state after the initial creation moment.

- Idle micro-motion keyed by mood: Why: visible behaviors like scan, preen, head-tilt, fluff, and alert movement make mood legible without meters.

- Top bar icons that fade after idle: Why: account, a11y, notebook, offer, and settle remain reachable while "no aviary-internal chrome" keeps the scene quiet.

- Day/night, settle, and weather transitions: Why: slow palette lerp, evening lean, call attenuation, and rain particles create a continuous local-time aviary rather than modal screens.

- Reduced-motion mode: Why: reduced motion still keeps "full sim + audio/captions" while removing leaf drift and using cross-fades, honoring both preference and setting toggle.

### Audio pipeline

- Procedural synthesis from motif graphs: Why: procedural calls avoid sample-pack fallback and recorded libraries while keeping species and individual offsets recognizable.

- Per-call seed parameters and captions mapped one-to-one: Why: captions can describe the actual produced call, such as prose from the live grammar.

- Buffer-pool reuse: Why: memory must stay flat over a 30-minute session.

- Listen-in gain ramps: Why: target-bird focus should happen gradually, with others ducked but "never mute."

- Evening/night mix decay with nightjar exception: Why: the day cycle quiets the aviary at night while preserving the nocturnally active species.

- WebAudio fallback to silence plus forced captions: Why: no audio should not make the product helpless; captions carry the call surface.

- First-gesture autoplay unlock and queued greeting: Why: browser autoplay policy may block sound, so visual-only remains viable until a gesture.

### Accessibility surfaces

- Screen-reader live region with naturalist paragraphs: Why: screen-reader users should receive "living prose" rather than coordinate or mood-enum dumps.

- Bird focus descriptions: Why: focused birds should expose name and observation so keyboard and screen-reader users can know individual birds.

- Notebook as accessible list: Why: notebook observations are a core surface and must be reachable through accessibility semantics.

- Call captions near birds: Why: captions translate live grammar into prose and fade with the call, making procedural audio perceivable without sound.

- Keyboard navigation for top bar, birds, listen-in, offer, and settle: Why: all core surfaces must be fully keyboard-operable.

- High-contrast focus rings and WCAG AA chrome/copy/captions: Why: contrast must hold across bright and dim skies.

- Matter-of-fact voice on system surfaces and naturalist voice elsewhere: Why: the product distinguishes operational copy from observational aviary prose.

### Performance and observability

- Initial JS, first-bird, FPS, memory, and tick p99 budgets: Why: the plan treats performance as part of v1 success on mid-tier 4G, older laptops, 30-minute sessions, and worker operations.

- Code-splitting, procedural media, small snapshots, CDN static, and snapshot parallel to shell paint: Why: these choices are "driven by budgets."

- Synthetic geo browsers and RUM aggregates: Why: load, first-bird, long tasks, and audio errors should be measured operationally without account/bird dimensions.

- Worker tick, event lag, and lease metrics: Why: the server simulation must remain healthy and observable.

- No per-bird product analytics: Why: per-bird funnels, streaks, and average-boldness analytics would violate the privacy/product stance.

### Rollout

- Internal dogfood with two fixed birds and drift instruments dashboards: Why: dogfood validates drift in ops-only dashboards "not user-facing numbers."

- Closed beta with magic-link allowlist: Why: the beta should validate "presence honesty vs laptop-asleep false positive."

- GA v1 with gates live and visits off by default: Why: general release preserves age-gated adoption and quiet optional visits.

- Birds-per-aviary ramp: Why: hard cap stays seven, most early users stay at two or three birds, and the cap should not rise without listen tests.

- Day-one instrumentation: Why: tick health, auth fail rates, snapshot latency, first-bird RUM, audio fail, memory, and drift harness protect launch quality without product analytics.

- Content ops freeze and tiny first-boot quiet field assets: Why: species art and motif libraries need a freeze checklist, and the first boot should remain small and quiet.

### Risks, mitigations, testing, and explicit UX rules

- Drift too fast mitigation: Why: per-day caps and weekly calibration prevent "Tamagotchi feel."

- Drift too slow mitigation: Why: weekly delta instrumentation and weight tinting prevent "screensaver."

- Presence honesty mitigation: Why: triple-gating and tests for hidden tabs/no input prevent "inflated population drift."

- Client vector write ban and mutation audits: Why: they prevent "silent personality loss" from dual-device merge bugs.

- Audio entropy audits and human listening: Why: procedural audio must avoid "uncanny / loops" and preserve aliveness.

- Greeting seeding and stagger: Why: identical greetings would feel like "theater."

- Accessibility same-release mitigation: Why: shipping narration, RMO, and captions together prevents a11y as an afterthought.

- Welcome toast ban: Why: welcome toasts are named as a "principle break."

- Bundle budget CI gate and asset accounting: Why: bundle bloat threatens TTFB and first-bird targets.

- Notebook no-LLM v1 and sparse rule templates: Why: the plan fears "generic spam" and "LLM-temptation."

- Visit APIs without comments or discovery: Why: the feature must not become a "social network."

- Soft-delete hard-delete checks and email expunge policy: Why: incomplete deletion is a "privacy breach."

- Autoplay-blocked mitigation: Why: captions and a quiet first click prevent "silent helpless."

- Unit, property, client, soak, and golden notebook tests: Why: tests protect monotonic drift, event-order safety, presence, reduced motion, keyboard, memory/audio stability, lowercase prose, and the ban on streak language.

- No tests asserting user-visible "level up": Why: levels are outside the product and would violate the anti-gamification surface.

- Explicit product UX rules: Why: no welcome banners, idle watching as interaction, no distress art, voice split, and living first pixel directly enforce the system-level product principles.

- Success criteria: Why: success is defined as knowing birds over weeks by ear and perch habits, same mood continuum across devices, a11y living prose, zero gamification, quiet visits, performance budgets, and truthful offline simulation.
