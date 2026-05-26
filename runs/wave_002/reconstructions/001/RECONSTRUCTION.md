## System-level intent

- **Feels alive through restraint and specificity, not announcement.** The plan states the defending phrase directly: "feels alive, notice never announce, charm from specificity, restraint." It shows up in "no wake-up animation, no spinner; quiet field on slow load," "sparse icons" that "fade on cursor still," sparse notebook entries, and exclusions like "no gamification," "no Tamagotchi mechanics," "no notifications/push," and no social-network surfaces.

- **Presence is real interaction, not mere absence of neglect.** The plan says "presence = real interaction (three-signal conjunction)" and defines presence as `visibilityState=visible + window focus + pointer/key activity`. It also says drift is "driven primarily by presence-time," that the "strict conjunction definition" prevents "tab open" corruption, and that visitor time gives "no drift contribution."

- **Drift should be monotonic toward expressive while protecting absence.** This appears in "Drift monotonic toward expressive," "monotonic expressive only," "No negative drift on neglect -> ambient quietness," and the risk note that the "monotonic rule protects absence." The plan wants drift to be measurable after "~1wk regular use" and visible after "~3wks," but not to become a hunger, distress, death, or happiness-decay system.

- **The server owns the living state; clients render and submit events.** The plan repeats "Server owns canonical aviary state + personality vectors + simulation tick," "Clients never write vectors," and "No client-owned personality state." This appears again in "single canonical record," "no merge," "no eventual consistency," and "prevents last-write-wins corruption of drift."

- **Privacy boundary strict, with social contact deliberately limited.** The plan uses "synthetic UUID," "encrypted email (one place)," "aggregate-only telemetry respecting privacy boundary," "no per-bird/per-account data in telemetry," and visits that are "opt-in, read-only, revocable." It also excludes profiles, feeds, discovery, leaderboards, comments, public aviaries, and co-presence.

- **Product voice is naturalist; system surfaces are matter-of-fact.** The plan says "Naturalist voice on product surfaces; matter-of-fact on system surfaces." It applies naturalist prose to the field notebook, screen-reader narration, and call captions, while errors and unsupported-browser pages are "matter-of-fact" and "never naturalist."

- **Accessibility and performance are v1 product requirements, not deferred cleanup.** The plan says "Ship with all listed surfaces; no deferred a11y/perf," "first-class designed surfaces," "reduced-motion register," WCAG AA, keyboard nav, captions, and strict budgets such as "<2MB gzipped bundle," "<500ms to first bird," and "60fps idle."

## Per-feature whys

### Scope

- **Single horizontal browser-based aviary scene with 2-7 birds:** The plan ties this to a "one-screen scene," "no panning/scrolling/zooming," and all birds visible. The cap of 7 also supports "recognizability" in the audio/chorus system.

- **Personality vectors and mood states:** The plan uses them to make birds expressive through drift and to shape mood, perch zone, motion, call timing, pitch, plumage saturation, and curiosity without exposing personality numbers.

- **Procedural call grammar:** The plan wants "real-time variation," "recognizability across drift," and a chorus that avoids an "uncanniness / canned feel." It explicitly says "No recorded loops ever."

- **Monotonic personality drift driven primarily by presence-time:** The rationale is "presence is real interaction," with presence as the primary signal and listen-in/offer/settle as secondary signals. Drift stays expressive only, with "No negative drift on neglect" so it does not become Tamagotchi-style decay.

- **Return-greeting (one bird):** NOT RECOVERABLE FROM PLAN

- **Listen-in:** The plan makes it both a secondary drift signal and an audio surface: "focus up, others ambient not silent," with a "slow rise/fall" mix ramp.

- **Offer seed/song/pool:** The plan connects offers to simulation and mood: "offer accepted -> content," "offer reaction," and offer events as secondary signals for drift/mood.

- **Offer cooldown:** NOT RECOVERABLE FROM PLAN

- **Settle gesture:** The plan connects settle to "quiet calls," a "slow evening shift," a 5-second undo, and secondary interaction input without scores, badges, or decay.

- **Field notebook:** The notebook gives sparse "naturalist prose" observations, about "1 every few days," without exposed state numbers. It shares the product voice with screen-reader narration.

- **Magic-link email sign-in:** NOT RECOVERABLE FROM PLAN

- **Synthetic UUID:** The plan uses a synthetic UUID as the account primary key, keeps encrypted email in "one place," and names it again as mitigation for "sync correctness / personality loss."

- **Single aviary per account:** NOT RECOVERABLE FROM PLAN

- **Visit invitations:** Visits are "opt-in, read-only, revocable," "default off," and "no drift contribution from visitors." This keeps visits inside the privacy boundary and outside social-network mechanics.

- **Multi-device sync:** The reason is a "server-side canonical state" that avoids client-to-client sync, merge, eventual consistency, and "last-write-wins corruption of drift."

- **Screen-reader narration:** It is an accessibility surface in the same "naturalist prose" as the notebook. The plan specifies "event observations not state lists" and priority for greetings, offers, and settle.

- **Reduced-motion mode:** The plan describes it as a "designed surface," not a "stripped fallback": cross-fade poses and perches, slowed color shifts, removed leaf drift, while retaining audio, captions, notebook, and drift.

- **WCAG AA contrast, keyboard nav, and call captions:** These are first-class accessibility requirements. Captions are "runtime-generated short naturalist prose per actual call," and keyboard access covers the top bar, birds, listen-in, offer, and settle.

- **Performance budgets:** The plan says the <2MB bundle drives "procedural audio, small SVGs/procedural visuals, aggressive code-split," while first-bird, 60fps, and no-memory-growth budgets keep the aviary immediately present and stable.

- **Server-side simulation tick:** The tick keeps canonical state moving about once a minute, consumes the event log, updates vectors and moods, and writes snapshots. It is "always running" even when the tab is hidden.

- **Presence accounting:** The plan gives the why as preventing "tab open" from counting as presence. The "strict conjunction definition" is visibility, focus, and recent pointer/key activity.

- **Aggregate-only telemetry:** The rationale is "privacy boundary strict": request counts, timings, errors, frame times, audio errors, and tick latency are allowed, but "no per-bird/per-account data."

- **Day/night and ambient weather:** The plan uses local time and weather to shape palette, mood transitions, and calls: "drowsy dusk," "alert morning," and "rain damps vocal freq."

- **Leaf/feather micro-motion:** The plan frames these as ambient client-generated ornaments that help the scene feel alive while staying outside canonical simulation state; reduced-motion removes leaf drift.

- **Adoption with two starter birds from a 6-species pool:** The plan says there is "no catalog selection for starters" and birds "arrive," which supports restraint and avoids a chooser/catalog surface.

- **User naming and renameable birds:** NOT RECOVERABLE FROM PLAN

- **Stable bird IDs:** The plan uses stable IDs to preserve identity across account state, snapshots, sync, export, and drift/personality persistence.

- **Naturalist voice on product surfaces and matter-of-fact system surfaces:** The rationale is voice containment: notebook, captions, and narration can be naturalist, while errors and unsupported states are "never naturalist."

### Architecture

- **Client/server split:** The reason is that the server owns "canonical aviary state + personality vectors + simulation tick," while clients stay "thin renderers" that pull snapshots and submit events.

- **Simulation, auth, snapshot, notebook, and visit services:** NOT RECOVERABLE FROM PLAN

- **Render pipeline boundary:** The client receives bird positions, moods, call timings, active transitions, day/night, and weather, then interpolates 60fps motion between snapshots. Pure rendering ornaments stay client-side, and rendering stops when hidden while simulation continues.

- **Append-only interaction events:** The reason is to let the tick consume ordered events and update vectors/moods server-side; "Clients never write vectors."

### Data model

- **Encrypted email in one place:** The plan's rationale is the privacy boundary around identity: synthetic UUID as primary, email encrypted in one place.

- **Visit log:** NOT RECOVERABLE FROM PLAN

- **Five personality scalars: boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity:** NOT RECOVERABLE FROM PLAN

- **Current mood, mood timers, and perch zone:** The plan uses these to make behavior visible: perch zone is chosen by mood/personality, and mood timers carry state through tick updates.

- **Presence event with a longer recent-activity window:** The plan says to "calibrate ~few min, favor longer," supporting real interaction while avoiding brittle presence loss.

- **Interaction event log per bird and per account:** It feeds the per-account simulation while staying "never aggregated beyond per-account sim," preserving the privacy boundary.

- **Notebook entry timestamp and sparse prose string:** The sparse cadence keeps the notebook observational and restrained rather than a continuous feed.

- **Visit invitation expiration of 30d:** NOT RECOVERABLE FROM PLAN

- **Snapshot payload:** The plan calls for current per-bird and aviary state in a "small payload," supporting thin clients and minimal critical path.

### API surface

- **Snapshot API:** The snapshot endpoint gives the client current state as a "small payload," matching the edge snapshot/minimal critical path performance plan.

- **Events API:** The events endpoint is append-only so interaction payloads can be consumed by the tick and not mutate personality state directly.

- **Notebook API:** NOT RECOVERABLE FROM PLAN

- **Export JSON:** NOT RECOVERABLE FROM PLAN

- **Delete with soft 30d then hard:** The plan later names "soft-delete recovery" as part of sync/personality-loss mitigation.

- **Visit read-only snapshot view:** The visit endpoint has "no event writes," so visitors cannot alter drift, mood, or canonical aviary state.

- **Matter-of-fact errors:** Errors are "never naturalist" to preserve the voice boundary between product surfaces and system surfaces.

### Simulation engine design

- **Tick consuming recent event log:** The tick centralizes presence, interactions, mood transitions, additive deltas, and canonical snapshot writes.

- **Drift function with low-pass filter and 1wk/3wks targets:** The rationale is calibration: too fast creates a "Tamagotchi feel," too slow feels "lifeless."

- **No negative drift on neglect:** The plan says absence becomes "ambient quietness," and the monotonic rule "protects absence."

- **Mood transitions from events, local time, weather, and personality:** The plan uses these to make birds responsive without gamified needs: offer accepted becomes content, dusk drowsy, morning alert, rain damped, high-bold less wary.

- **Call grammar runtime:** The runtime combines motif library, personality-shaped timing/pitch, mood modulation, procedural WebAudio, and live variation to keep calls recognizable but not canned.

- **Bird-to-bird call response, mood spread, and chorus emergence:** The rationale is "chorus emergence" and interaction among birds without social-network mechanics.

- **Mood persistence:** The plan says mood "carries across sessions + tick interim," giving continuity beyond a single client session.

### Sync model

- **Single canonical record per aviary:** The plan's reason is no client-to-client sync, no merge, no eventual consistency, and no last-write-wins corruption.

- **Snapshot pulls on visibility change, long gaps, and keepalive:** The plan uses these pulls to keep clients aligned with the canonical record after hidden tabs or long gaps.

- **Rare sync conflicts as matter-of-fact errors:** The rationale is the same system-surface voice boundary: errors stay matter-of-fact.

- **Revocable client session tokens:** NOT RECOVERABLE FROM PLAN

### Frontend rendering pipeline

- **Load places birds mid-action:** The plan avoids "wake-up animation" and "spinner," giving a "quiet field on slow load" so the aviary feels already alive rather than announced.

- **Responsive compress/widen preserving aspect and all birds visible:** This supports the one-screen scene and no panning/scrolling/zooming constraint.

- **Subtle parallax foreground/background:** The plan presents this as subtle scene depth inside the single horizontal aviary.

- **Idle micro-motion shaped by mood:** Wary scans back, content preens, curious tilts, and drowsy fluffs low; the why is making mood legible through behavior.

- **Perch changes with interpolated paths or reduced-motion crossfade:** The plan preserves movement continuity while honoring reduced-motion as a designed surface.

- **Top bar with sparse icons fading on cursor still:** The plan's rationale is restraint: controls stay available but visually quiet until activity returns.

### Audio pipeline

- **Procedural WebAudio synthesis:** The plan uses it to avoid recorded loops, keep the bundle small, and prevent "canned feel" while supporting per-bird/species motif libraries.

- **Listen-in gradual rebalance:** The plan says "focus up, others ambient not silent," so listen-in narrows attention without muting the living field.

- **Settle quiets calls:** The plan makes settle an atmospheric downshift, not a score or state decay mechanic.

- **Fallback with graceful silence and captions default-on:** The plan keeps the experience accessible if WebAudio is unavailable.

- **Call captions fading with calls:** Captions reflect the "actual call" as short naturalist prose, connecting accessibility to the live procedural audio.

### Accessibility surfaces

- **Screen-reader cadence of 30-60s idle:** The plan makes narration slow and observational, with event priorities, so it is not a state-list dump.

- **Reduced-motion cross-fades:** The plan explicitly says reduced motion is "not stripped fallback," keeping the full product surface.

- **Keyboard navigation through top bar and birds:** The rationale is complete keyboard access for the major interactions: arrows, enter listen-in, escape, offer, and settle.

- **High-contrast focus outline on aviary background:** The plan's rationale is visible focus over the visual scene, aligned with WCAG AA and keyboard nav.

### Performance budgets and observability

- **Bundle under 2MB gzipped:** The budget drives procedural audio, small SVGs/procedural visuals, and code-splitting for settings/visit flows.

- **TTFB under 500ms:** The plan connects this to "edge snapshot" and a "minimal critical path."

- **60fps idle for 30min with no memory growth:** The plan names buffer reuse, bounded contexts, and no retained references as the reasoned implementation path.

- **Synthetic browsers and RUM aggregate:** The rationale is to watch load timings, frame times, audio errors, and tick latency while preserving "no per-bird/per-account data."

- **Older-browser unsupported page:** NOT RECOVERABLE FROM PLAN

### Rollout

- **v1 includes starter birds, accounts, interactions, notebook, visits, a11y, tick, and sync:** The plan says "Ship with all listed surfaces; no deferred a11y/perf," so v1 is complete rather than staged around accessibility or performance.

- **Gradual bird-cap ramp:** The plan says to "instrument drift/recognition first," using recognition and drift signals before increasing the cap.

- **Day-1 instrumentation:** The plan allows aggregate request counts, latencies, render timings, error rates, and anonymized session-duration histograms, while again excluding per-bird telemetry.

### Risks and mitigations

- **Drift calibration mitigation:** The plan's why is explicit: too fast creates a "Tamagotchi feel"; too slow feels "lifeless."

- **Sync/personality-loss mitigation:** Server-only additive deltas, event order, synthetic UUID, soft-delete recovery, and no client mutation protect personality state.

- **Audio mitigation:** Strict procedural WebAudio, no loops, cap at 7, and chorus mixing address "uncanniness / canned feel."

- **Accessibility mitigation:** First-class narration, reduced-motion register, and no ARIA-only fallback avoid accessibility regressions.

- **Presence-signal mitigation:** Strict conjunction and a test harness prevent "tab open" from counting as real presence.

- **Voice-leakage mitigation:** Explicit non-goals, the matter-of-fact exception, and notebook generator enforcement prevent announcements, toasts, and streak language from leaking into the product voice.
