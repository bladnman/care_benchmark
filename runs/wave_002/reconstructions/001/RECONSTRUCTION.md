## System-level intent

- **Feels alive through continuity rather than spectacle.** This shows up in "server tick continuity," "mid-action first frame," "procedural calls," "idle motion," "tick continues transitions while user away," and "bird already mid-preen/call." The plan wants aliveness to be ambient and persistent, not introduced by an "intro animation" or a "blank-then-fade."

- **Notice never announce.** The plan repeats this through "Bird greeting only," "no toasts/streaks," "No toast, no 'welcome back,' no days-gone text," "never unison fanfare," and "forbid toast lib on aviary route." The product should let birds respond without system messages announcing the user's behavior back to them.

- **Charm from specificity.** The plan ties charm to "Notebook & narration templates," "derived observable facts," "specificity," "no generic achievements," and "Template review; sparsity; ban systemy phrases." It prefers naturalist observations such as greeter order, perch, fluffed state, or call density over abstract logs.

- **Restraint.** The plan names "7 bird cap, one scene, sparse chrome, calm palette" and excludes streaks, badges, XP, follows, public discovery, chat, comments, leaderboards, co-presence, payments, multi-aviary accounts, and customizable scenes. The experience is meant to stay small, quiet, and uncrowded.

- **Dual voice.** Product-facing surfaces use "Naturalist product surfaces," "naturalist screen-reader narration," "field notebook," "observation voice," and "voice continuity." System-facing surfaces use "matter-of-fact system," "matter-of-fact errors," "Matter-of-fact labels," and "visit no longer available."

- **Presence as attention.** Presence is the "drift spine" and is protected by "triple-condition," a "4 minutes" activity window that "favors 'watch without moving'," "presence samples," and "server daily caps." The plan explicitly rejects "tab-open-alone" and "no invented presence."

- **No Tamagotchi.** The plan keeps "Monotonic drift," "ambient quiet on neglect," "no downward trait," "no negative drift," and excludes "death, hunger, distress, decaying meters." Long absence changes "short-term greeting intensity" through recency, not stored traits.

- **Server-authoritative canonical state.** The architecture principles say "Single writer for personality/mood canonical state: simulation tick only," "Clients are render + eventsources," "Server authoritative," "no LWW personality," and "ban client trait fields." The plan treats client-owned simulation as a direct risk.

- **Privacy and data minimization around identity, traits, and operations.** This appears in "Synthetic account UUID," "email encrypted," "Personality raw values excluded from snapshot," "not in normal client API," "Simulation DB isolated," "Ops telemetry only," "no bird ids, no traits, no per-user interaction content," and "no per-bird warehouse."

- **Accessibility as a peer surface, not retrofit.** The plan requires "first-class accessibility," "Ship a11y with v1 visual path," "not as lagging retrofit," "Reduced-motion" as "Own aesthetic, not 'broken static'," "call captions," keyboard support, and WCAG AA chrome.

- **Performance discipline serves the first bird.** The plan centers budgets around "<2MB gzip," "TTFB bird <500ms," "Time to first bird visible," "60fps idle," "no mem growth 30m," "quiet field" loading, hidden-tab rAF cancellation, AudioContext suspension, buffer pools, and soak tests.

- **Limited sociality.** Visits are "optional read-only visit invites," "off by default," "revocable," per-invite email, and separate from the rejected "Social network." Visitor tokens have "events API 403" and "visitor attention cannot drift host birds."

## Per-feature whys

### Scope and v1 commitments

- **Modern web platform:** NOT RECOVERABLE FROM PLAN

- **Browser-only, single-account canonical aviary:** the plan ties this to "one canonical aviary per user," "one scene," and the principle of "Restraint." Multi-aviary accounts and native apps are out of scope.

- **Email magic link auth:** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions:** NOT RECOVERABLE FROM PLAN

- **One horizontal scene with 3 perch zones:** the plan grounds this in "Restraint" and in rendering requirements such as "Three perch anchors responsive width" and "never crop birds."

- **Day/night local-time:** the plan uses local time to drive both surface and simulation: "day/night local-time," "Day/night from client local tz applied to palette," and "server also uses account tz for mood."

- **Rare weather:** the plan uses weather for ambient aliveness and behavior: "rare weather," "Advance weather RNG," "Rain" mood effects, and notebook boosters for "weather + behavior."

- **Two starter birds from a species pool:** the plan says starters are server-assigned as a "diverse pair," supporting immediate aviary life without a catalog browse.

- **Seven-bird cap:** the plan names the cap under "Restraint" and delays sixth/seventh rollout until "audio lab validates recognizability at 5-7."

- **Stable bird IDs:** NOT RECOVERABLE FROM PLAN

- **Server-owned simulation and tick:** the plan's rationale is canonical continuity: "Server authoritative," "Single writer," "tick continues transitions while user away," and no "client-owned sim."

- **Personality vector plus mood:** the plan uses these to create "presence-driven monotonic personality drift," "mood transitions," greeter choice, perch preference, plumage shifts, and call behavior.

- **Procedural calls client-side:** the plan prefers "procedural calls," "client synthesizes samples," and "no recorded MP3 path" to keep calls recognizable, varied, lean, and not dependent on recorded-audio fallback.

- **Return-greeting:** the why is "Notice never announce." It replaces toast copy with bird response: "No toast, no 'welcome back,' no days-gone text."

- **Presence:** the plan treats presence as attention and the "drift spine." The strict measurement prevents "silent population over-drift" and rejects "tab-open-alone."

- **Listen-in:** the plan makes listen-in a focused attention signal that affects "social_warmth" and "vocal_frequency" while preserving ambience: the mix is rebalanced, "not mute."

- **Offers:** the plan uses offers as positive interaction signals: offer accept affects "curiosity," offer proximity affects "boldness," and reactions depend on "mood x curiosity."

- **Settle:** settle "ends presence cleanly," has "Delta = 0 for traits," creates a quieter snapshot, and makes tab close equivalent to "presence end without penalty."

- **Field notebook:** the rationale is "Charm from specificity" and naturalist voice. Entries are sparse, use "derived observable facts," and avoid raw traits or "you visited N days."

- **Read-only visit invites:** the plan keeps sociality limited: visits are "read-only," "off by default," "revocable," and visitor attention "cannot drift host birds."

- **JSON export:** the plan frames this as "Data rights" and later as "Trust," with the full personality vector available only through export "by user request."

- **Soft delete for 30 days then hard delete:** the plan also frames this under "Data rights" and "Trust," with cancel available "within 30d."

- **First-class accessibility surfaces:** the plan says to ship a11y "with v1 visual path, not as lagging retrofit," including narration, reduced motion, captions, keyboard, and WCAG AA chrome.

### Ambiguity calls

- **Four-minute presence activity window:** the plan gives the rationale directly: it "favors 'watch without moving'."

- **Sixty-second nominal tick cadence:** NOT RECOVERABLE FROM PLAN

- **Trait range as [0.0, 1.0] floats:** NOT RECOVERABLE FROM PLAN

- **Mood enum including settled:** the plan gives "settled for night/settle" as the reason for that mood value.

- **Template-plus-slot notebook assembler:** the plan prefers it "for predictability and privacy."

- **Age-based bird-3 and later adoption timing:** the plan says adoption is "never visit-count gated," preserving the anti-gamification stance.

- **Presence pings every 30s:** the plan uses them so the "server aggregates duration" while the triple condition holds.

### Architecture, data, and API

- **API gateway / BFF with separate auth, simulation, DB, CDN, and ops telemetry services:** NOT RECOVERABLE FROM PLAN

- **Synthetic account UUID and encrypted email:** the plan's rationale is minimization: "Synthetic account UUID everywhere except one encrypted email column."

- **Simulation DB isolated from analytics and ops telemetry:** the plan states "network + IAM deny," "Ops telemetry only," and "no per-bird warehouse," separating operations from simulation data.

- **Client/server split for personality, mood, positions, calls, presence, notebook, and visits:** the why is server authority plus thin clients: clients "pull snapshots" and "append interaction events," while the server stores drift, transitions mood, generates notebook entries, and strips visit writes.

- **Client-only ornament layer:** NOT RECOVERABLE FROM PLAN

- **Hidden-tab behavior:** rAF cancellation, AudioContext suspension, presence off, and re-pull on visibility restore support performance and prevent hidden tabs from counting as attention.

- **Raw personality excluded from normal snapshots:** the plan says raw traits are "never sent," "not in normal client API," and excluded to prevent "Personality leak to UI" and "Stat optimization."

- **Append-only InteractionEvent with idempotency:** the plan uses event order for sync: concurrent sessions append events and the tick "serializes by event id time"; idempotency supports retries.

- **Notebook preview count without content in snapshot:** NOT RECOVERABLE FROM PLAN

- **Generic auth responses and matter-of-fact errors:** the plan's product-voice reason is "matter-of-fact system," including "Matter-of-fact errors only" and matter-of-fact magic-link timeout copy.

- **Event API rejects personality and mood writes:** the plan protects the "Single writer" rule, rejects "absolute personality/mood writes," and bans client trait fields.

- **Offer cooldown clamps:** NOT RECOVERABLE FROM PLAN

- **Visitor snapshot path with write APIs forbidden:** the plan says visitor routes are read-only, return "events API 403," and must prevent "Visit presence bleed" because it would leave the "Host relationship distorted."

### Simulation engine design

- **Offline tick without invented presence:** the plan gives the rationale directly: "no invented presence."

- **Monotonic upward drift with ambient quietness on neglect:** the plan uses this to deliver "No Tamagotchi," avoiding downward trait loss while letting long absence lower greeting intensity through recency.

- **Weighted presence preference energy:** the rationale is calibrated positive attention: "Presence dominates," listen-in strengthens social and vocal traits, offers affect curiosity and boldness, and settle contributes no trait delta.

- **Low-pass trait update:** the plan says the update should "asymptote at 1 without overshoot drama."

- **Drift calibration targets:** the plan wants change to be detectable after "7 days" in replay, user-visible around "3 weeks," and "below perceptual threshold" for a single session.

- **Mood transitions:** mood bias from time, weather, offers, neighbors, and personality keeps birds changing while the user is away and supports "Feels alive."

- **Call grammar with species motif and per-bird jitter:** the plan says "Recognizability > variety" while preserving individual call signatures.

- **Limited simultaneous callers and listen-in mix floor:** the plan prevents "Audio uncanny / looped feel" and keeps listen-in as "mix, not mute."

- **Return-greeting absence buckets and stagger:** the plan makes greeting proportional to absence, chooses a likely greeter, and avoids "unison fanfare."

- **No welcome toast or days-gone text:** the plan says those would violate "notice-never-announce" and "destroys notice-never-announce."

- **Adoption without catalog browse:** the rationale is "birds that arrived," with age thresholds only and "adoption unlock clocks real."

- **Notebook token bucket, boosters, and observable facts:** the plan uses sparsity plus "derived observable facts" to avoid "generic logs," raw trait numbers, and visit-count narration.

### Sync, frontend, audio, accessibility, performance, and rollout

- **Same snapshot version across devices and refetch on restore or long gaps:** the plan uses this for multi-device sync and for "snap interpolate from new targets without teleports if possible."

- **No last-write-wins personality; ordered event serialization:** the rationale is avoiding "Lost weeks of self."

- **Hard re-base after huge snapshot gaps:** the plan states the preference as "science over jitters," quietly prioritizing canonical state over smoothness.

- **Quiet-field loading, no spinner, soft retry, and no write-toast spam:** the plan preserves the calm field voice and avoids announcing system state inside the aviary.

- **Top bar fade and sparse chrome:** the plan ties this to "sparse chrome" and "Restraint"; chrome stays outside the scene and fades after idle input.

- **Reduced-motion mode:** the plan treats this as inclusion with charm: "Own aesthetic, not 'broken static'," while keeping color phase, calls, and captions.

- **First frame aliveness:** the rationale is explicit: hydrate pose and phase so the bird is already mid-action and "Defeat default SPA blank-then-fade."

- **Audio silence plus captions fallback, with no recorded MP3 path:** the plan keeps a graceful no-audio path and respects the non-goal of recorded-audio fallback.

- **Mute preference still counts presence:** the plan says muting is "not negative drift," so audio preference must not punish attention.

- **Narration, captions, keyboard, focus ring, contrast, and settings:** the plan wants "observation voice not ARIA state dumps," captions from the same grammar, keyboard access, AA contrast, and "Matter-of-fact labels."

- **Performance budgets, code-splitting, procedural assets, buffer pools, and soak tests:** the plan uses these to protect first-bird time, bundle size, 60fps idle, and flat memory over 30 minutes.

- **Aggregate-only observability:** the plan permits TTFB bird, FPS proxy, audio errors, timing, long tasks, and API latency, while forbidding bird IDs, traits, per-user interaction content, engagement streaks, and raw drift dashboards.

- **Internal dogfood, closed beta, and GA phases:** NOT RECOVERABLE FROM PLAN

- **Bird-cap ramp with feature flag:** the plan uses the ramp to validate call "recognizability at 5-7" before enabling sixth/seventh globally.

- **Day-one instrumentation and CI tests:** the plan uses these to catch tick lag, ingest errors, first-bird timing, audio failures, magic-link failure modes, drift calibration problems, memory growth, snapshot personality leaks, and presence-condition regressions.

- **Content and design dependencies:** the rationale is to support the intended product voice and aliveness: "calm palette," "vocabulary docs for naturalist vs matter-of-fact," and "species art + motif libraries."
