## System-level intent

1. **Charm is correctness, not polish.** The plan opens by saying "the product's charm is not separable from its correctness"; a "canned greeting," "leaked numeric trait," or "harmless toast" is a "P1 bug." This intent shows up again in the testing, linting, DTO, prose, and rendering rules.

2. **The design principles are acceptance criteria.** In §1, the PRD's five design principles and non-goals are "not a preamble" but "the acceptance criteria." §3 is called "the spine," and if a later decision conflicts with §3, "§3 wins."

3. **Feels alive, not robotic.** G1 requires the first frame to be "mid-motion," calls that "never repeat identically," and procedural, mood-keyed idle motion. The same principle drives the first-frame bootstrap (§9.1), idle micro-motion (§9.3), procedural audio (§10), and return-greeting variation (§7.5).

4. **Notice, never announce.** G2 says there is no welcome toast/banner/modal and that "the bird greeting is the entire welcome." This appears in the return-greeting as bird behavior only (§7.5), the absence of toast/notification infrastructure (§3), and the no textual welcome tests (§17.1).

5. **Charm from specificity.** G3 requires "naturalist, bird- and moment-specific prose," never generic state language or gamification. The notebook (§7.11), narration (§11.1), captions (§11.2), and `sim-core/prose` voice system (§12) all carry this.

6. **Restraint over richness.** G4 keeps the product at "2 birds start, 7 cap; one screen; no in-scene chrome; calm palette." The scene (§9.2), top bar (§9.5), species cap (§7.10), and no gamification/social surfaces (§2.2) all enforce restraint.

7. **Naturalist voice and matter-of-fact system voice are deliberately split.** G5 and §12 define `prose.naturalist.*` for aviary surfaces and `copy.system.*` for auth, settings, errors, sync, and accessibility settings. The plan says any surface where the user engages with the system "as a system" drops out of naturalist voice.

8. **Presence must be an honest signal.** G6 defines presence as visible AND focused AND recent pointer/key activity. §7.2 calls this "the honest signal" and forbids the "tab is open" shortcut because it would "corrupt drift population-wide."

9. **Personality is hidden, expressive, and server-owned.** G7 blocks numeric exposure, G8 makes the server tick the only personality writer, and §5.4 limits snapshots to "expressions" and render buckets. §8.3 says clients send events like "listened in to Pip for 3 min," never "set boldness = 0.62."

10. **Drift is monotonic toward expressive, never punitive.** G11 and §7.3 require no negative-delta path for absence. The plan says neglected birds become "ambient," never "wary/silent/duller," and that "absence simply adds nothing."

11. **Slow truth belongs to the server; fast presentation belongs to the client.** §4.1 names this the "load-bearing split": the server decides personality, mood, day phase, call disposition, perch intent, and weather windows, while the client handles sub-second presentation. This lets devices converge without a sync channel.

12. **One deterministic implementation keeps surfaces aligned.** The shared `sim-core` exists so captions match audio, narration matches visuals, and visitor and host views are identical (§4.1, §7.6, §8.2, §19-A1).

13. **Privacy is architectural absence, not policy text.** G9 isolates email as PII; G10 says per-bird/per-account interaction data is never aggregated. §13.5 and §14.2 emphasize that visit-frequency, days visited, and metrics that could later build a streak or leaderboard are "deliberately NOT measured / NOT computed."

14. **Accessibility is a designed v1 surface.** G12 says narration, captions, reduced-motion, keyboard navigation, and contrast ship with v1. §11 rejects a "fallback" accessible experience and §15.1 says accessibility surfaces are present from the start.

15. **Affective performance protects the product conceit.** §13.2 calls the 500ms first-bird budget the "affective-perf bridge"; if the user notices a load, the "already running" conceit breaks. The same principle drives no spinner (§9.1), 60fps idle (§13.3), and no memory growth (§13.4).

16. **Bird identity and personality are relationship state.** §5.6 says bird ids are immutable and vectors are stored, never rebuilt, because vector loss is "deleting the relationship." §15.4 and §16 treat personality loss / sync corruption as the worst failure.

## Per-feature whys

### Scope, accounts, identity, and lifecycle

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN.

- **Email magic-link sign-in and consumption:** The route behavior is justified by "no account-existence oracle": `POST /auth/magic-link` always returns 202, with per-email rate limits. Consumption is bounded by "≤15m" and "unconsumed" single-use semantics.

- **Per-device revocable sessions:** NOT RECOVERABLE FROM PLAN.

- **Email change with verification:** NOT RECOVERABLE FROM PLAN.

- **Account export JSON:** The articulated rationale is to provide meaningful account state without breaking G7: exports include current mood and "coarse plumage bucket," not raw scalars, so the product never turns a bird "into a number" (§6.6, §19-A7).

- **Soft-delete for 30 days, then hard-delete:** The plan says soft delete allows sign-in to recover within 30 days; hard delete removes "birds, vectors, notebook, telemetry tied to the account, all gone" (§14.3).

- **One canonical aviary per account:** NOT RECOVERABLE FROM PLAN.

- **Stable bird ids and stored personality vectors:** A `bird.id` is "allocated once and is immutable," and vectors are "stored, never recomputed" because this is "the foundation of perceived drift validity"; loss equals "deleting the relationship" (§5.6).

- **Synthetic UUIDs and encrypted email:** Email is PII and appears only once, encrypted, while `account_id` is used everywhere else. §14.1 says this is honored at design time because it is "impossible to retrofit."

- **Per-account settings for reduced_motion, captions, audio_on, visit_notify, and timezone:** NOT RECOVERABLE FROM PLAN.

### Architecture, data, and sync

- **Stateless `api` service:** It handles auth, snapshots, events, settings, export/delete, visits, and account management while holding "no per-session simulation state," making it "horizontally scalable" (§4.1).

- **`sim` worker:** It "owns the tick" and is the sole writer of personality vectors, moods, and notebook entries, preserving the server-only personality rule (§4.1, G8).

- **`web` browser client:** It "renders only" and is "never authoritative for personality," which keeps slow truth server-owned (§4.1).

- **Shared `sim-core`:** The plan calls this a deliberate choice because it keeps "captions matching the audio actually played," narration matching visual state, and visitor view identical to host view (§4.1, §19-A1).

- **Render-pipeline boundary:** The server decides "slow truth"; the client decides "fast presentation." This prevents the client from inventing canonical state and prevents the server from dictating per-frame motion (§4.1).

- **Snapshot polling, not WebSocket/SSE:** The plan says the tick is slow and clients interpolate, so push is unnecessary. Polling is "simpler, cheaper, CDN-friendly," and snapshots are only kilobytes (§4.2, §19-A2).

- **Simulation store:** Postgres is the "canonical record" with strong consistency and isolation from analytics, supporting canonical state and G10 (§4.3).

- **Object store for account-export blobs:** NOT RECOVERABLE FROM PLAN.

- **CDN/edge initial HTML shell with inlined minimal snapshot:** This serves the first paint and supports drawing the first bird quickly before non-critical assets load (§4.3, §9.1, §13.2).

- **Analytics warehouse:** It exists only for "aggregate operational telemetry" and has no network route to the simulation store, preserving G10 (§4.3, §14.2).

- **TypeScript end-to-end:** The stated reason is that TypeScript makes shared `sim-core` real (§4.4, §19-A1).

- **Preact + signals for chrome UI:** It is "small enough for the bundle budget" while expressive enough for settings, notebook, offers, auth, and visit chrome (§4.4, §19-A3).

- **Canvas2D scene renderer behind a `SceneRenderer` interface:** Canvas2D is sufficient for "≤7 birds + a few parallax planes" and avoids shader/bundle weight, while the interface keeps a WebGL backend possible if instruments demand headroom (§4.4, §19-A4).

- **AudioWorklet synthesizer:** It runs synthesis off the main thread so render load does not create scheduling glitches (§10.1).

- **Opaque random auth tokens hashed at rest:** NOT RECOVERABLE FROM PLAN.

### Simulation engine and interactions

- **Composability invariant and lazy catch-up tick:** The plan reconciles "continue without the viewer" with scale by requiring `advance(state, t0, t1)` to match many sub-steps. Recently active accounts tick continuously; dormant accounts catch up lazily, and both paths agree exactly (§7.1).

- **Presence detector:** Presence is visible, focused, and recently active because "watching birds without moving is the actual product." The long activity window avoids treating still watching as absence, while forbidding "tab is open" as presence (§7.2).

- **Drift function:** Drift is the "slow spine." Presence, listen-in, and offers create positive evidence; absence has no negative branch. Calibration targets keep a single session invisible, one week instrument-detectable, and three weeks user-visible (§7.3).

- **Mood FSM:** Mood is the fast-timescale state over interactions, time of day, ambient events, and personality. It persists across sessions so tab open never snaps mood to neutral; night sleep is distinct from user-triggered `settled` lighting (§7.4).

- **Return-greeting:** It is "the anchor moment" and the whole welcome. Absence length, boldness, warmth, mood, seeded variation, and staggered greeters prevent the feature from collapsing into "play arrival animation" or a textual announcement (§7.5).

- **Call-grammar runtime and seeded scheduler:** The server sets call disposition and the client schedules onsets deterministically so host devices, visitor view, and captions converge without realtime sync. Variation prevents identical calls; the motif core keeps bird signatures recognizable (§7.6).

- **Day/night from account timezone:** Local-time anchoring is required so "the user's morning is the aviary's morning." Pure `(tz, server_now)` derivation keeps every device and lazy catch-up reproducible (§7.7).

- **New-bird availability:** Offers are age-gated only so the mechanic never teaches "more attention earns more stuff." It is presented as "a bird has arrived," not as a catalog (§7.8).

- **Settle with 5s undo:** Settle creates evening lighting, quieter calls, and drowsy birds; undo is "a mercy for misclicks, not a feature." Settle and tab-close are equivalent presence endings and neither is penalized (§7.9).

- **Species pool, adoption, naming, and no rarity:** Species are chosen to read as "one coherent place." Starters are selected by the system as "the birds that arrived"; naming has no engine effect and there is no species rarity (§7.10).

- **Notebook generation:** The notebook is "sparse by construction" so entries do not become noise. It records observations of the aviary, never the user's behavior, to avoid becoming a disguised streak (§7.11).

- **Seed / song fragment / still pool offer types:** NOT RECOVERABLE FROM PLAN.

- **Offer cooldown:** The cooldown is "functional (prevents within-session curiosity saturation that would collapse the engine), not punitive," and makes an offer read as "a gesture, not a button-mash" (§7.12).

- **Ambient weather:** Weather is deterministic from seed and wall-clock so clients and lazy catch-up agree. It is rare, short, and "never assertive," with only short-lived mood nudges (§7.13).

- **Bird-to-bird interaction:** Calls, wary spread, and chorus make the aviary "a small social system, not a row of independent NPCs" (§7.14).

### API and visit flow

- **Snapshot DTO with expressions, not raw traits:** The DTO includes mood enum, perch, plumage bucket, and call disposition, but never raw trait scalars, enforcing G7 at the client boundary (§5.4, §6.2).

- **Append-only interaction events:** Users can only append events; the tick processes them in `id` order with `consumed_tick` idempotency. This removes last-write-wins and prevents clients from sending personality values (§5.3, §6.3, §8.3).

- **Bird rename endpoint:** NOT RECOVERABLE FROM PLAN.

- **Visit invite, read-only visitor session, and unmodified visitor snapshot:** Visit is the only read-only social affordance. The visitor receives the host's current snapshot "unmodified," with no show-off rendering, no visitor drift, revocation, expiry, and matter-of-fact unavailable copy (§6.5).

- **Host visit log:** It is on-demand and limited to visitor email, date, approximate duration, and outstanding invites, fitting the plan's "silent by default" visit posture (§2.1, §6.5).

- **Opt-in visit notification:** It is the only optional notification, "off-by-default," preserving the no notification surface while allowing a per-visit "a friend visited" setting (§2.1, §2.2).

- **Visit revoked/expired/used surface:** It uses matter-of-fact copy because revoked, expired, and unavailable visit states are system surfaces under G5 (§6.5, §12).

### Frontend rendering and audio

- **First-frame bootstrap and quiet loading field:** The aviary must appear "already in motion." There is no spinner because "a spinner says machine"; the quiet field is used only if the snapshot is slow (§9.1).

- **One horizontal scene with three perch zones:** One screen, no pan/scroll/zoom, and no cropping preserve restraint. Perch zones encode proximity, and because birds choose perch from mood/personality, perch is a signal the user reads (§9.2).

- **Idle micro-motion:** Motion is continuous, procedural, and mood-shaped so the scene never reads as paused or cycle-locked. Mood is "read from motion," with no labels, tooltips, or status icons (§9.3).

- **Transitions:** Perch flights, greeting stagger, settle ramp, day/night interpolation, weather fades, and leaf/feather drift preserve continuity while keeping ambient ornaments client-side rather than server state (§9.4).

- **Top bar:** The top bar is "the only chrome," with exactly four icons and no badges or notifications. It fades nearly transparent so the scene remains restrained (§9.5).

- **Render lifecycle:** Rendering pauses when hidden to save battery, while the server keeps ticking. On return, the client pulls current truth rather than resuming from a frozen pre-hide frame (§9.6).

- **Reduced-motion render mode:** It is "a different rendering of the same aviary," calmer and slower, not broken or stripped. Calls, captions, drift, mood, and notebook remain unchanged (§9.7, §11.3).

- **Procedural WebAudio:** Calls are synthesized, never downloaded, because procedural synthesis enables variation, real chorus, and the bundle budget; recorded audio fallback is a hard non-goal (§10.1).

- **Per-call variation and signature recognizability:** Variation ensures no two calls are identical, while a stable motif signature makes "knowing Pip from Wren by ear" possible and supports the 7-bird cap (§10.2).

- **Chorus:** Realtime procedural voices avoid stacked recorded loops that "phase-cancel audibly," and chorus emerges from the bird-to-bird model (§10.3).

- **Listen-in mix:** Listen-in gradually raises one bird and lowers others toward an ambient floor, "never to silence," because it is a re-balance rather than mute/solo. It is also a strong attention signal feeding drift (§10.4, §7.3).

- **WebAudio fallback:** If WebAudio is unavailable, the fallback is "graceful silence with captions on by default" because "silence + captions beats canned audio" (§10.5).

- **Audio memory discipline:** Reused buffers, voices, and bounded contexts support the no-memory-growth CI test (§10.6, §13.4).

### Accessibility and voice

- **Screen-reader narration:** Narration uses naturalist prose from the same canonical state as visuals, so screen-reader users hear "one product." The slow cadence avoids flooding the SR queue (§11.1).

- **Call captions:** Captions are generated from the same call-grammar parameters as the played call, so the caption "matches the call actually played" (§11.2).

- **Keyboard navigation:** It makes the top bar, scene birds, listen-in, offer affordance, and settle reachable through keyboard paths, fulfilling accessibility as a first-class v1 surface (§11.4, G12).

- **WCAG AA contrast:** The plan sets AA as the floor for user-copy text, with automated contrast checks on chrome tokens (§11.5).

- **Naturalist prose namespace:** `prose.naturalist.*` is lowercase, present-tense, bird- and moment-specific, with no "you," announcement framing, or gamification words. It powers notebook, narration, captions, offer prompts, and aviary-surface copy (§12).

- **System copy namespace:** `copy.system.*` uses normal capitalization and direct language for sign-in, settings, errors, accessibility settings, unsupported browser, and visit-revoked surfaces. It avoids "warmth-as-evasion" where the user is engaging the system as a system (§12).

### Performance, observability, privacy, rollout, and testing

- **Initial JS bundle under 2MB gzipped:** The budget is enforced in CI and drives code-splitting, procedural/small visuals, and procedural audio (§13.1).

- **Time-to-first-bird under 500ms:** This is release-blocking because over 500ms the user notices a load and the "already running" conceit breaks (§13.2).

- **60fps idle over 30 minutes:** It is a runtime budget, not just first minute, so the aviary remains alive and stable during extended watching (§13.3).

- **No memory growth over 30 minutes:** It is a CI test, not a guideline, because leaks across rendering, notebook DOM, workers, or audio would degrade the long calm session (§13.4).

- **Aggregate-only observability:** The plan measures operational health while deliberately not measuring per-bird state, per-account interaction history, visit-frequency, or anything that could later become a streak or leaderboard (§13.5).

- **Browser support:** Last two major versions are supported; older browsers get matter-of-fact unsupported-browser copy, and very old compatibility paths are rejected because bundle cost is not justified (§13.6).

- **Telemetry boundary:** Per-bird interaction events exist only to drive the user's own simulation, never aggregation, training, recommendations, third-party sharing, or population analysis (§14.2).

- **Privacy policy link in settings:** It names aggregate categories and explicitly excludes per-bird state, matching the telemetry boundary (§14.3).

- **Rollout phases:** Internal alpha proves engine, calibration, perf, and accessibility early; closed beta exercises multi-device sync and aggregate drift distribution; GA turns on bird-ramp and visit while visit remains off by default per account (§15.1).

- **Server-side calibration flags:** Tick cadence, drift constants, presence window, mood timers, weather frequency, and bird-age thresholds are server-side so calibration can be tuned from aggregate data without client redeploys (§15.3).

- **Migration safety:** Migrations preserve bird identity and vectors verbatim, forbid rebuilding from logs, and take vector backups before touching `bird` because vector loss is the worst product failure (§15.4, §16).

- **Testing and calibration strategy:** The tests exist to make guardrail violations caught in CI or review: greeting variation, no textual welcome, voice denylist, monotonic drift, no identical calls, sync correctness, presence honesty, privacy lints, perf gates, accessibility checks, and DTO raw-trait guards (§17).
