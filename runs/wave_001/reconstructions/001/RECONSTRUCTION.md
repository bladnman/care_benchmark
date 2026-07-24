## System-level intent

- Prefer notice-over-announce and sparsity over feature density. This is named directly in Scope as "prefer notice-over-announce, sparsity over feature density" and appears again in the boot rule "no spinner," the success bar that the user never sees "spinner-of-machine" or "Welcome back," notebook "sparsity," and the risk called "Spinner / welcome toast prince creep."

- Keep the product non-gamified and non-Tamagotchi. The plan excludes "streaks, badges, levels," "XP," "counters," "death, hunger, distress, decaying happiness," and "push/email about aviary state." The same intent returns in drift: "Monotonic toward expressive only," "neglect does not decrease traits," adoption unlocks "never engagement score," and the risk "Drift too fast | Tamagotchi feel."

- Make the server the owner of canonical personality and relationship state. The architecture says "Clients never tick and never write personality," the split says "Server owns" personality vectors, mood, drift, cooldowns, notebook, and ACL, and sync repeats "No client personality mutation" and "No LWW on vectors."

- Make continuity across absence and devices feel real. The plan ties snapshots, `motion_phase`, return-greeting, background return, and multi-device sync to "birds feel continuous across absence and across phone/laptop." It also requires "canonical server state + append-only interaction event log" and one `GET snapshot` for all devices.

- Express aliveness through procedural, recognizable behavior rather than canned assets. This shows up in "procedural calls," "AudioGrammar," "Motif library per species," "Recognizability invariant across mood," "independent generators mixed, not loop stack," and the risk "Canned audio / loops sneaks in | Kills aliveness."

- Make accessible paths the same charming product, not a stripped-down state dump. Accessibility is described as "screen-reader narration (naturalist prose)," "call captions," "reduced-motion designed surface," "Same voice as notebook," "Not ARIA 'mood: content' dumps," and the stance "accessible paths stay charming, not checklist shells."

- Keep privacy and analytics restrained around relationships. Scope excludes "per-bird interaction data in aggregate analytics / ML"; telemetry is "Aggregate ops only"; the plan says not to store "visit patterns as social graphs for ranking"; and the success criteria says "Ops can see health without reading anyone's relationship."

- Use matter-of-fact system copy for system states, not naturalist voice. API errors are "matter-of-fact copy," visitor revoke gets a "matter-of-fact 410 body," unsupported browsers get a "matter-of-fact upgrade message," and sign-in/session/load failure strings are "no naturalist voice."

## Per-feature whys

### Scope

- Browser-only SPA: NOT RECOVERABLE FROM PLAN

- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN

- Synthetic UUID account ID: The plan wants account paths keyed by a synthetic UUID because "email never used as partition/key outside account table" and the email-as-ID risk is "PII sprawl."

- Single-user accounts with one aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds and age-gated offers up to seven birds: The plan makes adoption "pure aviary age thresholds" and "never engagement score," while the hard cap supports the success constraint that "Seven-distinct call recognizability remains design constraint."

- Server-side simulation tick, personality drift, mood, and bird-to-bird behavior: The plan assigns these to the server so clients "never tick and never write personality," avoiding "Client LWW / optimistic personality" and "Silent identity loss."

- Single horizontal scene: NOT RECOVERABLE FROM PLAN

- Day/night using user local timezone: The plan uses account TZ for "Time-of-day" mood biases and a local-time palette, so morning, dusk, night, and night-active exceptions can shape the aviary.

- Ambient weather: Weather is a rare scheduler with "few rains/week; soft wind," nudges mood, dampens vocal expression, and can provide a notebook "rare spark" when noteworthy.

- Procedural calls: The plan bans "Recorded-audio fallback path," "sample packs," "multi-MB sample library," and "3-clip rotation" because canned audio and loops "Kills aliveness."

- Return-greeting: Greeting uses "boldness x mood x absence_length" after last presence end so return behavior reflects absence without a generic "Welcome back."

- Presence accounting: Presence is the dominant drift signal, but it is clamped and credited as "unique wall-clock minutes" so open tabs and dual devices do not "double-speed drift."

- Listen-in: Listen-in focuses on a bird by ramping that bird up and others down "but never mute," and its duration influences `social_warmth` and `vocal_frequency`.

- Offer: Offers are resolved server-side by "curiosity/mood near gesture zone" and drift `curiosity` and `boldness`, while cooldowns prevent concurrent offers from becoming client-side personality writes.

- Settle: Settle "end presence cleanly" with "no special drift direction," forces an "evening wash," and keeps the feature from becoming a punishment or reward mechanic.

- Field notebook: The notebook is for rare, noteworthy "lowercase naturalist" prose; the plan enforces "sparsity" and immutable entries instead of a dense log or stat surface.

- Multi-device sync through canonical server state and append-only event log: This preserves one canonical aviary across devices; all sessions pull the same snapshot and events are ordered before the tick writes state.

- Optional visit invitations: Visits are "off by default," "read-only ambient," and "revocable" because the plan resists social-network "Gravity shift" and says visitor presence gives "no presence contribution to host drift."

- Accessibility bundle: Screen-reader narration, reduced motion, captions, contrast, and keyboard nav exist so a11y users get "naturalist aliveness, not inventory UI."

- Account export: Export is justified as "ownership/portability"; raw vectors may appear there while the UI still never shows them.

- Soft-then-hard delete: NOT RECOVERABLE FROM PLAN

- Session revoke: NOT RECOVERABLE FROM PLAN

- Visit log in settings: NOT RECOVERABLE FROM PLAN

### Architecture and data model

- Auth service: NOT RECOVERABLE FROM PLAN

- API gateway: NOT RECOVERABLE FROM PLAN

- Simulation worker: The worker owns the "~60s" tick loop, folds events into canonical state, schedules ambient weather, and triggers notebook generation so simulation is not run by clients.

- State store with append-only `interaction_events`: The append-only event log lets the tick process ordered interactions and prevents personality from being last-write-wins.

- Realtime / snapshot path with optional SSE/WS: Realtime push is optional because correctness comes from snapshots; the plan says SSE/WS is "not required for correctness" and "correctness never requires WS."

- Mailer: NOT RECOVERABLE FROM PLAN

- Telemetry: Telemetry exists for "Aggregate ops only" and is "separate from simulation DB" so health can be observed without relationship data.

- Client/server split: The server owns personality, mood, drift, cooldowns, notebook, and ACL; the client owns rendering, WebAudio, presence detection, local palette, and caption layout so cosmetic interpolation cannot mutate canonical simulation.

- Render pipeline boundary: Snapshot JSON flows into scene, animation, audio grammar, and narration, while "No game loop" advances canonical simulation; the client only interpolates waypoints and ornaments.

- Account `email_enc` and `email_hash`: Email is encrypted or hashed for lookup because the hard rule says email is never a partition/key outside the account table and the risk is "PII sprawl."

- Account timezone: Timezone supports "Time-of-day" mood biases and the local day/night palette, with fallback to client-reported offset.

- A11y prefs and visit notify settings: These store captions, reduced-motion override, and visit notify opt-in so accessible paths and optional visits are explicit user choices.

- Session device tokens: NOT RECOVERABLE FROM PLAN

- Aviary `version`: Version is used for "snapshot races" so clients can discard stale optimistic UI.

- Bird stable identity: Bird IDs are "stable identity forever" and the hard rule says identified birds have "immutable identity across rename."

- Server-only personality vector: Raw personality numbers are stored server-side and never shown because the product rejects "Numerical personality exposure" and "Stat-management temptation."

- Mood, placement, and cooldown fields: These fields support snapshot hydration, perch targeting, mid-action continuity, and server-side cooldown enforcement rather than client mutation.

- Call grammar seed: The immutable `call_seed` preserves "motif identity" and the call recognizability invariant across mood.

- Species catalog and night-active flag: Species carry silhouettes, palettes, motif libraries, and night-active exceptions so time-of-day behavior can differ by species.

- PresenceEvent buckets: Presence buckets let the tick fold presence-seconds with "anti-inflation rules" instead of treating every ping or device as full credit.

- NotebookEntry prose and internal source signals: Notebook entries surface "lowercase naturalist" prose while keeping `source_signals` internal only, maintaining voice without exposing mechanics.

- VisitInvite, VisitSession, and VisitLog: Visit records support revocable, expiring, read-only guest access and explicitly avoid contributing visitor presence to host drift.

- MagicLink expiry and consumed state: NOT RECOVERABLE FROM PLAN

### API and interaction paths

- Auth consume, sign-out, and email-change endpoints: NOT RECOVERABLE FROM PLAN

- Account paths keyed by synthetic UUID from session: This enforces the account rule that all account paths are keyed by synthetic UUID and not by email.

- Snapshot full render payload: Snapshot includes birds, weather, lighting, time phase, version, and `server_now` so the client can render and hydrate without owning simulation.

- Snapshot omits raw personality vector numbers: The plan says "Omit raw personality vector numbers from client forever" and allows only derived presentation fields to avoid numerical personality exposure.

- Snapshot pull triggers: Pulling on `visibilitychange`, long rAF gaps, and focused keepalive lets the app hydrate after absence or backgrounding with current canonical state.

- Batch append `/aviary/events` with idempotency keys: Events are stamped server-side and do not apply personality directly, keeping interaction writes reliable but outside the personality mutation path.

- Server-side offer recipient resolution and cooldowns: The server resolves recipients by curiosity, mood, and gesture zone, and server-side cooldowns make concurrent offers ordered rather than last-write-wins.

- Presence endpoint triple-condition plus clamp: The client sends only visible+focused+recent_input and the server clamps rate to prevent "Presence inflated by open tabs" and population overdrift.

- Notebook reverse-chrono cursor endpoint: NOT RECOVERABLE FROM PLAN

- Adoption endpoints: Adoption is unlocked by aviary age rather than engagement score, preserving the anti-gamification stance.

- Settings CRUD: NOT RECOVERABLE FROM PLAN

- Export endpoint with email download link: Export supports user "ownership/portability."

- Delete and delete-cancel endpoints: NOT RECOVERABLE FROM PLAN

- Host visit invite, list, revoke, and log endpoints: Host visit APIs keep visits opt-in and revocable, resisting social-network gravity.

- Visitor token snapshot: Visitor access is read-only, can work as a guest token without an account, and posts no events except optional telemetry count.

- Matter-of-fact error copy: Errors use matter-of-fact language because sign-in, timeout, load failure, revocation, and browser upgrade are system states, "no naturalist voice."

### Simulation and sync model

- Tick cadence with shard jitter: Jitter exists "to avoid thundering herd," and tick p99 has an alarm threshold.

- Idempotent event processing: Processing events by server timestamp in the last tick window makes ticks repeatable and ordered.

- Presence buckets in the tick: Buckets convert qualifying presence into clamped presence-seconds so drift reflects watching without open-tab overcredit.

- Weather scheduler: Rare rain and soft wind provide ambient variation, mood nudges, vocal dampening, and possible notebook sparks without becoming dense content.

- Time-of-day mood biases: Account TZ drives morning alert, dusk drowsy, night settle, and night-active species exceptions to make behavior feel time-bound.

- Bird-to-bird wariness and chorus: Threshold wariness contagion and chorus windows let birds respond to each other, making the scene social without a social network.

- Mood transitions: Mood changes combine personality modifiers, recent offers, listen-in, and rain so state changes arise from both character and recent context.

- Perch targets: Boldness, wariness, and social warmth map to front/back/cluster tendencies so personality is expressed spatially without exposing numbers.

- Slow LPF drift that is monotonic toward expressive: Drift is tuned so regular presence is measurable after about a week, visible after about three weeks, invisible in one session, and neglect does not punish.

- Pose and `motion_phase` advance: Advancing pose and phase supports "snapshot mid-action" and continuity when hydrating after gaps.

- Notebook candidate selection: Notebook generation is a "rare spark" for noteworthy events and capped at about "2-3/week baseline" to enforce sparsity.

- Drift calibration harness and property tests: Tests confirm 0 presence does not move traits, heavy presence accelerates within bounds, and neglect paths do not decrease vectors.

- Call grammar runtime: Species motifs plus personality pitch/timing jitter keep calls recognizable across mood while avoiding loop stacks and sample libraries.

- Greeting selection: Greeters are chosen from boldness, mood, and absence length, with staggered secondary greetings and procedural variety so returns feel personal without clip rotation.

- Adoption unlock age thresholds and hard cap: Unlocks use real-world aviary age, "never engagement score," and stop at seven for recognizability and non-gamified pacing.

- Canonical source for sync: One aviary row and birds per account make every device resolve to the same `GET snapshot`.

- Conflict prevention for names, settings, and optimistic UI: Versioning or row locks avoid rare conflicts, and clients discard stale optimistic UI to prevent "Silent identity loss."

- Multi-device presence credit: The plan credits "unique wall-clock minutes" with at least one qualifying device, not summed devices, so two devices do not speed drift.

- Background tab behavior: Background tabs stop render and presence credit, while the server continues ticking and the client pulls a fresh snapshot on return.

### Frontend rendering, audio, and accessibility

- Quiet field boot with no spinner: The boot paints a "quiet field" and forbids spinner/product-start animation so the user never sees "spinner-of-machine" or an announced return.

- Top bar fade and no chrome inside scene: The bar fades toward transparent after idle and restores on pointer/key, aligning with "notice-over-announce" and "No chrome inside scene."

- Responsive gaps that never crop birds: NOT RECOVERABLE FROM PLAN

- Continuous idle motion: Preen, scan, head-tilt, and weight shuffle are "mood-shaped," giving aliveness while the server keeps canonical control.

- Reduced-motion visual path: Reduced motion uses still pose sequences and slow cross-fades, no leaf drift, and a slow day palette so the path is designed, not a checklist shell.

- Day/night settle wash and undo: Settle forces an evening wash and allows undo within five seconds, making settle reversible and not a drift punishment.

- Perch interpolation and mid-action hydrate: Interpolated perch moves and mid pose/phase placement keep birds continuous between snapshots.

- Procedural plumage and code-split assets: Procedural plumage, compact sprites, and code-split drawers help keep the bundle under the JS budget and preserve first-bird timing.

- WebAudio synthesis and buffer reuse: Motif oscillators, envelopes, filters, and buffer reuse create calls without permanent allocation or sample packs.

- Listen-in mix ramps: The mix uses slow ramps to focus one bird while others remain audible, so listen-in is not a "channel switch."

- Captions from the same grammar as audio: Captions share the grammar instance with audio, keeping silent or captioned output aligned with the generated call.

- No-WebAudio fallback silence plus captions: The fallback is silence with captions default on because the plan rejects a recorded-audio fallback and sample packs.

- Small motif bundle with no sample library: Compact motif params and no multi-MB sample library protect the "<2MB gzip JS budget."

- Screen-reader live region: A polite live region with naturalist running prose gives "naturalist aliveness" instead of ARIA mood dumps or inventory UI.

- Keyboard navigation: Keyboard flow through the top bar, birds, listen-in, and exit exists for "full keyboard nav."

- Contrast: WCAG AA chrome, captions, and system UI protect readability across bright and night scenes.

- Captions toggle and auto-on silent path: Captions can be user-controlled and automatically enabled when audio is unavailable, matching the "silence+captions" stance.

### Performance, observability, and rollout

- Performance budgets: The bundle, first-bird, frame-rate, memory, and tick targets preserve mid-mobile load, long idle sessions, and operational health.

- Aggregate measurement set: TTFB, first-bird paint, hitch rate, audio errors, tick latency, HTTP errors, and anonymous duration histograms are measured to see health without account-dimensional relationship data.

- Deliberately unmeasured data: The plan avoids per-bird vectors, identity-tied offer contents, visit ranking graphs, and "engagement optimization" funnels to prevent gamification and privacy drift.

- Privacy plane separation: Simulation DB is not joined to analytics warehouse for bird fields, preserving the boundary around relationship data.

- Internal dogfood rollout: Dogfood limits scope to two birds, tick, presence, greetings, audio, and quiet load so the core aliveness and drift loop can be tested first.

- Closed beta rollout: Closed beta adds multi-device, notebook, settle, and the a11y triad once the core loop exists.

- v1 public rollout: NOT RECOVERABLE FROM PLAN

- Ramp birds-per-aviary: Bird count ramps by real-world weeks, monitors call recognizability usability comments, and holds at seven.

- Day-one instrumentation: Tick health, snapshot latency, first-bird timing, audio fails, drift canaries, and privacy sampling tests verify the product's core promises early.

- Feature flags for weather density, notebook sparsity, and unlock ages: Server-side flags allow tuning ambient density and pacing, explicitly "not gamification toggles."

- Browser gate: Unsupported user agents get a matter-of-fact upgrade message, matching the system-copy stance.
