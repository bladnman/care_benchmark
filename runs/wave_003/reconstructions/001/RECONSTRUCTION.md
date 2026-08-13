## System-level intent

- **A relationship with a window.** The opening says, "The product is a relationship with a window," and the closing says, "Ship a window that was already going." This intent shows up again in the architecture section: "The split is the product," because the aviary must continue without the viewer and must not feel like a client-side toy that starts when the user arrives.

- **Server-canonical continuity.** The plan repeatedly makes the server the owner of meaning: "Clients never tick and never write personality," "Devices are projectors," and the worker is "the only process allowed to UPDATE birds personality/mood/perch columns." The why is continuity across absence and devices: if laptop and phone each simulated, "return would require merging two climates."

- **Personality is additive and non-punitive.** Binding decisions say "Personality is additive server-authored deltas only" and "Drift is monotonic toward expressive." The plan separates this from ambient quietness: neglect "does not lower traits" and instead uses a "recency-weighted expression layer." The product intent is change through presence without punishment.

- **Presence must be measured, not guessed.** Presence is defined as "the conjunction of visibility + window focus + recent pointer/key activity," and a laxer definition is "a product bug." The same principle appears in presence union across devices, the 240 second activity window, and the statement that the "Three-signal conjunction cannot be reconstructed later."

- **No announcement, score, punishment, or neediness.** The plan forbids "welcome toast, streak, badge, visit ping," "achievements, streaks, levels, scores," and metrics that would make meetings about "making birds needier." Notebook prose cannot praise attendance, visits are not badges, and new birds are not "you earned a bird."

- **Voice split: naturalist for the aviary, matter-of-fact for systems.** The binding decision says "naturalist on product surfaces; matter-of-fact on auth, errors, account, sync, and accessibility settings." This shows up in notebook and narration prose, while auth/errors reject phrases like "the flock could not find you."

- **Accessibility is a designed surface.** The plan says "Accessibility ships in v1 as designed surfaces, not fallbacks." Reduced-motion is a renderer swap, narration and captions are first-class modes, and a11y work is scheduled in M1/M2 with release blockers equal to FPS.

- **Privacy is pipeline shape.** The plan says "Privacy = pipeline shape." Email lives in "one encrypted column," per-bird history never enters aggregate telemetry, the simulation DB is not peered to the warehouse, and the plan explicitly rejects dashboards like "average curiosity."

- **Aliveness is the release gate.** The quality bar includes "First frame already in motion," "Quiet-field loading (never a spinner)," and "60fps idle." Rollout says M1 is "the aliveness spike" and "Do not launch M4+ if M1/M2 feel canned."

- **Identity continuity is sacred.** The plan calls out "Identity continuity of birds across rename, sync, and species-catalog changes." It protects this with immutable `birds.id`, immutable `call_seed`, stable species grammar, and no "regenerate aviary" admin action.

- **Procedural uniqueness over canned media.** The plan rejects recorded calls and canned greetings: "No recorded call audio, ever," "No sample files for calls," and "Avoids canned client-side 'play arrival #2'." Calls, captions, greetings, and motion are generated from seeds and grammar.

- **Quiet social boundaries.** Visits are optional, read-only, off by default, revocable, and stripped of action cursors. The plan names "Visit feature social pressure" as a risk and mitigates it with "no show-off render," notify off by default, and "no public lists."

## Per-feature whys

### 1. Scope

- **Browser-only SPA and supported browser set:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **Magic-link auth:** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** The plan ties this to server-canonical state and avoiding "merging two climates"; one aviary keeps continuity honest across devices and absence.

- **Two starter birds with different species and contrasting call families:** The starter pair is chosen so starters are "distinct but not extreme" and never "two trillers," giving identity and contrast without a catalog-picked start.

- **Cap of seven birds:** The cap is reflected in audio and product rules: the mixer is "built for 7," and simultaneous voices are capped "by product rule" so chorus and rendering stay bounded.

- **New birds unlock by aviary age, not attention:** The plan uses age thresholds to avoid attention-based rewards. The adopt affordance is "not 'you earned a bird,'" and unlocks write a notebook line rather than a login modal.

- **Server-side simulation tick:** The tick exists so the aviary "continues without the viewer"; mood, perch, weather, bird prompts, and notebook authorship are not dependent on a live client.

- **Single non-scrolling horizontal scene:** The scene supports the "window" product shape: no zoom, pan, or scroll, a letterbox-free layout, and a requirement to "never crop a bird."

- **Return-greeting:** The greeting expresses return without announcement UI. It is computed from absence, mood, boldness, and `greeting_readiness`; a low score produces "only a glance pose," which the plan says is valid.

- **Idle presence:** The plan treats watching as the product. The 240 second activity window leans long because "watching without moving is the product," and qualified presence drives slow personality drift.

- **Listen-in:** Listen-in is meant to be "a slow rebalance, never a mute." It opens an attention window, raises the focused bird while keeping others audible, and feeds warmth/vocal drift without turning listening into a solo mode.

- **Offers: seed, song, still pool:** Offers provide feedback through reaction, not numbers: "the reaction is the feedback." Accepted offers affect curiosity, nearby offers affect boldness, and cooldowns prevent offer mashing.

- **Settle with 5s undo:** Settle quiets the aviary without punishing or rewarding traits: it ends presence, moves mood quiet, contributes no trait direction, and undo clears settled without inventing presence.

- **Field notebook:** The notebook is server-authored, sparse, naturalist, and read-only so it feels like observation rather than a feed. "Active users do not get a feed," and entries avoid "you," counts, and trait numbers.

- **Rename:** Rename is allowed while preserving identity. The plan says birds maintain continuity across rename, and concurrent rename is the only LWW field because "names are not drift."

- **Multi-device sync:** Sync is a property of server-canonical state. Devices submit events and render snapshots so phones and laptops do not write competing personalities.

- **Optional visit invitations:** Visits are read-only, ambient, revocable, and off by default to avoid social pressure and show-off behavior. Visitors are not given event POST, offers, notebook, listen-in, or presence probes.

- **Account export and deletion:** Export is framed as a "user-owned copy" and excludes tokens, hashes, and raw event logs. Deletion is soft for 30 days, then hard-deletes personal state; deletion limbo pauses ticks so absence is not "time served."

- **Accessibility features in v1:** Accessibility is included because it is part of the designed product surface, not a fallback: narration, captions, reduced-motion, WCAG AA chrome, and full keyboard path all ship in v1.

- **Performance budgets in v1:** The budgets serve aliveness: first bird visible under 500ms, 60fps idle, flat heap, and initial JS under 2MB make the first frame and ongoing window credible.

- **Product name and internal service names:** "Pocket Aviary" and `aviary-*` service names support the plan's refusal to frame the product as `pet-*` or `game-*`.

- **First-frame motion, quiet-field loading, procedural greetings, sparse notebook, identity continuity quality bar:** These are in scope as polish because the plan treats them as the product's feel: "First frame already in motion," "never a spinner," "Procedurally unique greetings," and stable bird identity.

### 2. Architecture

- **TypeScript Hono API gateway:** TypeScript owns the HTTP edge "because the client already speaks it," keeping contracts close to the browser and product surfaces.

- **Go sim-worker:** Go owns the tick because "tick latency, lease correctness, and 'never skip an account silently'" are easier in a small dedicated worker than request-scoped Node.

- **No Kafka, no warehouse, no ML feature store in v1:** The event log is a Postgres table, operational metrics are aggregate, and "Simulation rows are not replicated into analytics," preserving privacy and scope.

- **Client writes events while server appends and interprets them:** Clients "report what happened, not what it means." This prevents client-authored personality and keeps simulation meaning server-owned.

- **Greeting selection precomputed into the snapshot:** The worker precomputes `next_greeting` to "avoid canned client-side 'play arrival #2'" and keep greetings procedural and contextual.

- **Canvas scene graph with Preact chrome:** React/Preact owns chrome, not bird poses, because if React owned poses "first-frame-mid-action and 60fps idle become accidental." Canvas gives the renderer control of frame state and motion.

- **Reduced-motion as renderer swap:** Reduced-motion is not a skipped animation flag because accessibility is first-class; the plan says it is "a renderer swap" and "not a 'skip rAF' flag."

- **Inline boot snapshot with allowed staleness:** The boot snapshot exists for time-to-first-bird. It may be up to 90s stale because mood/perch are "close enough" and a keepalive corrects within one RTT; first paint must not wait on a live tick.

### 3. Data model

- **Encrypted email, lookup hash, and synthetic account UUID:** Email is PII, stored in one encrypted column, and never used as a log key or partition key. UUIDs carry logs, keys, and traces.

- **Aviary `founded_at` as frozen age clock:** `founded_at` drives new-bird offers and is "not reset on recovery," preserving aviary age and avoiding deletion/recovery as a reset path.

- **`presence_minutes_7d` as expression-layer input, not personality:** The plan uses rolling presence for ambient expression, not trait reversal, so quietness can change while personality remains monotonic.

- **Stable bird IDs, stable `call_seed`, and reserved `retired_at`:** Bird identity is never recycled. `call_seed` "never changes," and `retired_at` is reserved so identity is not deleted except on hard account delete.

- **No API trait numbers except export:** The plan forbids trait numbers in UI because visible numbers would turn personality into a score surface; export is allowed because it is "a user-owned copy."

- **Expression layer distinct from personality:** `greeting_readiness` and `chorus_heat` may decay so neglect becomes "ambient quietness without lowering traits." Personality itself is not lowered.

- **Append-only events:** Events support a tick cursor, idempotency, and audit of what clients reported. They are not updated except `processed_at` and not deleted except hard account deletion.

- **Read-only notebook entries:** Notebook prose is authored by the worker, not the user, so clients cannot edit/delete/annotate it into a social feed or task list.

- **Species catalog as code with no rarity:** Six species form "one coherent temperate-woodland set." "Rarity is not a feature," which keeps species from becoming collection status.

- **Trait soft ceiling at 0.97:** The soft ceiling means drift never "completes"; traits can keep moving slowly without becoming a finished progress bar.

- **Snapshot exposes only needed trait-derived fields:** `plumage_saturation` reaches the renderer because color needs it, but it is "never labeled." Other traits influence pose, perch, and call params through tick-derived fields.

- **Export excludes session tokens, magic-link hashes, and raw event logs:** The export gives the user settings, birds, vectors, moods, names, notebook, and invite metadata while keeping credentials and raw event history out.

### 4. API surface

- **Rate limits for magic-link, event ingest, snapshot, and invites:** NOT RECOVERABLE FROM PLAN

- **Magic-link route returning the same 202 body whether or not the email exists:** NOT RECOVERABLE FROM PLAN

- **Matter-of-fact auth and account copy:** Auth and account are system surfaces, so they follow the mechanical voice split. Magic-link email copy has no "your birds miss you."

- **Idempotency on writes:** Idempotent events support flaky network replay and mid-write timeouts without a merge UI; event outbox replays use the same `client_event_id`.

- **Snapshot route with cache/etag/keepalive pulls:** Snapshots are how clients render canonical state while sparse server ticks continue. `If-None-Match`, boot, visibility return, and keepalive keep the window aligned.

- **No client `POST /personality`:** The plan states clients do not write personality because client-submitted trait values would break continuity and invite LWW conflicts.

- **Presence pings with three-signal qualification and 240s activity window:** The server accepts only well-formed pings because the three signals "cannot be reconstructed later." The long window protects passive watching.

- **Visit render-only API and stripped visitor snapshot:** Visitor snapshots remove greetings, offer action state, notebook, and write cursors so visits stay ambient and read-only.

- **Visit-notification email as opt-in, sparse, matter-of-fact:** The only visit notification fires at most once per visitor per 24h, has no bird names or duration, and uses the log as "the detail surface," avoiding badges and growth loops.

- **Error voice:** Error bodies are matter-of-fact and explicitly reject naturalist phrasing like "the flock could not find you," keeping system failures out of product fantasy.

### 5. Simulation engine

- **Transactional tick loop with cursor and snapshot refresh:** The tick locks the aviary, processes ordered events, writes birds/aviary, advances the cursor, marks events processed, and refreshes cache so canonical state and read snapshots stay aligned.

- **Ticks still advance with no events:** If no events are present, pose, lighting, and expression still advance because "The aviary is not frozen between visits."

- **Catch-up chunking after worker outage:** Virtual minutes are chunked and capped to bound transaction time, while mood still follows local sun during catch-up.

- **Drift low-pass, weights, and calibration targets:** The `(1 - x)` term makes early movement easier and late movement slow. Targets make drift "measurable in fixtures," "user-visible" around week 3, and not visibly step after one session.

- **Neglect behavior:** With neglect, `Delta x = 0`; traits hold while `greeting_readiness` decays. After two weeks away, greetings are "rare glances" while color and identity remain intact.

- **Mood as soft time-of-day attractor:** Mood is not a "midnight snap." It persists across sessions, mixes local time with recent events, and remains server-written.

- **Perch choice and users never placing birds:** Perches express boldness, mood, social warmth, and greeting boost through server choice. Clients render the resulting behavior rather than letting users arrange birds.

- **Bird-to-bird calls, wary spread, and chorus:** Bird-to-bird prompting makes birds respond to one another. Chorus is marked so the client mix "breathes rather than stacking peaks."

- **Return-greeting selection and absence bands:** One primary greeter is selected by boldness, readiness, and mood. Absence length changes from glance to staggered calls, preserving return feeling without unison or announcement.

- **Offer receiver, cooldown, and reactions:** A 4-minute per-bird cooldown and visual-only reactions when unavailable prevent rapid trait pumping. Accepted and nearby offers produce small deltas, but the visible reaction carries the feedback.

- **New-bird unlock and adopt affordance:** Unlocking by age avoids attention rewards. The affordance is a "small extra mark" rather than a modal or "you earned a bird."

- **Server/client split for call grammar:** The server emits identity-bearing grammar params and windows; the client micro-varies within them so devices are not "phase-locked in a disturbing way" while call identity remains stable.

- **Notebook author and sparsity controls:** The worker writes only on rare triggers with cooldowns so "Active users do not get a feed." Lints reject achievement, streak, visit, and trait-number language.

### 6. Sync model

- **Single writer with devices as projectors:** Devices post events to the log and read snapshots. There is "no client-to-client channel and no CRDT" because server state prevents personality conflicts.

- **Conflict prevention instead of resolution:** Personality cannot conflict because clients cannot write it. Offers and settles become event rules; only name LWW remains, because "names are not drift."

- **Presence union across devices:** Overlapping qualified intervals are unioned, not summed. The plan says this "protects calibration" when the user watches on two devices.

- **Client pull triggers and optimistic local visuals:** Pulls on boot, visibility return, sleep gaps, keepalive, and event posts keep snapshots fresh. Optimistic offer/settle visuals provide quick feedback while personality waits for the tick and must not appear to jump.

- **Interpolation between snapshots:** Movement eases over 8-14s and never teleports. If stale, the client places the new state mid-pose with no fade from black.

- **Offline outbox and no fake ticks:** IndexedDB replays unsent events, but the client does not invent ticks offline. If offline long enough, ornaments freeze low-energy and the server-canonical snapshot catches up later.

- **Magic-link/session race handling:** Used links immediately expire, events are idempotent, and failures produce matter-of-fact errors rather than a "pick a version of your birds" dialog.

- **Deletion versus sync:** Soft-deleted accounts show recover everywhere and pause ticks, because deletion limbo should not make absence count as "time served."

### 7. Frontend rendering pipeline

- **Layered single-canvas scene with no crop:** The layer stack creates the aviary window, while no zoom/pan/scroll and responsive layout ensure birds stay visible on phones and wide desktops.

- **Parameterized bird rigs instead of large bitmaps:** Rigs keep idle procedural and the "bundle stays small"; no sprite sheet is used for idle motion.

- **Idle micro-motion:** Birds maintain phase clocks and "Never zero velocity" unless reduced-motion, supporting the sense that the aviary was already moving before load.

- **First frame and audio unlock behavior:** Boot uses `pose_phase` so birds start mid-gesture. No-snapshot shows quiet field, and audio unlock waits for the first pointer/key without a big "click to enter" splash.

- **Lighting, weather, and ornaments:** Local-time lighting, rare rain/wind, settle palette, and client-only ornaments provide atmosphere while keeping weather server-authoritative and per-leaf state off the network.

- **Top bar fade, icon-only controls, and no badges:** The top bar fades after stillness and has no labels inside the scene, no badges, and no notification-like unread dot, reinforcing the no-announcement UI principle.

- **Offer flow as top-bar popover rather than click-on-bird:** NOT RECOVERABLE FROM PLAN

- **Settle top-bar, undo, and reengage behavior:** Settle quiets the scene with a 5s undo; after that, clicks are reengage, aligning interaction with quieting rather than scoring.

- **Reduced-motion renderer:** Reduced-motion uses still poses, cross-fades, no leaf drift, slower color shifts, and no blank fade-in because it is a designed rendering mode, not absence of rendering.

- **Calm naturalist color and high-contrast focus:** The palette has "No electric accents," while focus and copy must pass AA against noon and night skies, keeping visual voice calm and accessible.

- **Preact route table and no marketing site:** NOT RECOVERABLE FROM PLAN

### 8. Audio pipeline

- **WebAudio graph and bounded mixer:** The graph separates per-bird voices, ambient, song offers, chorus bus, and master gain so voices can overlap without limiter pumping.

- **Procedural voice with no samples:** Calls are generated from motif grains, species templates, call seed, and mood/personality shaping. This satisfies "No recorded call audio, ever" and avoids a "same file" feeling.

- **Recognizable bird identity:** Species rhythm skeleton plus stable `call_seed` means mood changes tempo/space, not identity; "A listener two weeks in should name Pip without a label."

- **Chorus jitter and allpass differences:** Independent jitter and per-bird allpass differences prevent grid starts, phase cancellation, and peak stacking.

- **Listen-in mix:** The 2.4s equal-power ramp and nonzero other-bird gain make listen-in a "rebalance" and avoid a DAW-solo feeling.

- **Autoplay unlock and fallback:** If audio is blocked, visuals continue without a modal. If WebAudio fails, the session uses "silence + captions forced on" rather than MP3 fallback.

- **Caption generation from the motif expander:** Captions are generated from the same grain scheduler, so they match calls, use naturalist phrasing, and are not a fixed map of strings.

- **Audio memory discipline:** Preallocated grain pools and no hot-path disconnect/reconnect protect the 30-minute flat heap requirement.

- **Settle, night, and rain audio density changes:** Call density drops for settle, night, and rain so audio follows mood, time, and weather without changing identity.

### 9. Accessibility surfaces

- **Accessibility tree, keyboard path, and focus overlay:** A hidden live region and roving-tabindex bird list prevent a canvas-only experience. The focus ring is mirrored visually on canvas.

- **Narration composer, cadence, and priority:** Narration is observational, sparse, and prioritized so it does not spam. It avoids trait numbers, "welcome back," and visit-frequency language.

- **Caption and reduced-motion settings:** Settings use matter-of-fact labels like "Call captions" and "Reduce motion"; system reduced-motion is honored immediately and then ORed with the account flag.

- **Contrast, names, and notebook document behavior:** Chrome, captions, errors, and notebook text meet AA. The notebook remains a real scrollable document so text is not dropped from the accessibility tree.

- **Visit accessibility:** Visitor view keeps narration and reduced-motion while presenting birds as non-interactive or skipping them, matching the render-only visit model.

### 10. Performance, observability, rollout, risks, testing, and team shape

- **CI-enforced budgets, code splitting, and small assets:** Budgets guard the first-bird and long-running window experience. Code splitting and path-data rigs keep the critical path below size and performance limits.

- **Synthetic and aggregate RUM measurements:** Measurements cover boot, first-bird mark, long tasks, audio errors, and snapshot RTT, but dimensions stay aggregate: browser, country, connection bucket.

- **Deliberately not measuring engagement or traits:** The plan rejects "Funnel 'engagement,'" "Average boldness," offer CTR, and visit conversion so success does not become making birds needier.

- **Privacy as pipeline shape:** DB/network boundaries, ETL rules, APM scrubbers, and UUID-targeted flags make privacy structural instead of copy-only.

- **M0-M7 rollout sequence and M1/M2 gate:** The sequence builds contract, aliveness, audio, simulation, interactions, sync, visits/a11y, then dogfood. M1/M2 must not feel canned because "Aliveness is the release gate, not feature count."

- **Bird-count ramp and staging-only acceleration:** Production starts with two birds, with the first third-bird wave about six weeks post-launch by construction. Staging can accelerate age for QA, and production cannot set that flag.

- **Launch instrumentation:** Day-one metrics are operational and aliveness-focused: first-bird timing, audio fallback, tick p99, presence-qualify ratio as ops health, JS errors, and magic-link consume success. Offer DAU is explicitly rejected.

- **Coarse feature flags:** Flags can disable visits, adoption, and weather. There is no flag for streaks "because streaks do not exist in the code."

- **Risk mitigations:** Each named risk ties back to product intent: drift calibration avoids Tamagotchi/screensaver extremes, sync correctness protects additive state, audio QA avoids canned loops, a11y blockers equal FPS, and privacy mitigations prevent leakage.

- **Testing strategy:** Unit, property, golden audio, visual, e2e, perf CI, and a11y CI make the plan executable and specifically test monotonicity, identity, reduced-motion, offer cooldown, visit revoke, heap, and tick behavior.

- **Monorepo and team shape:** The repo is split into web, api, sim, contracts, naturalist-copy, and species packages so vertical slices can be owned while contracts are reviewed by everyone. The plan says not to stand up a "growth" workstream.

- **What not to build:** Debug trait overlays, fake offline ticks, push, public "your bird grew" changelogs, and a game loop with scores are forbidden because everything is "in service of" the already-going window.
