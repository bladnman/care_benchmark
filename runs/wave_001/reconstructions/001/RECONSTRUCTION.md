## System-level intent

- Non-gamified, non-punitive aviary care. The plan draws a hard line around "no streaks, levels, scores, badges, achievements" and "no death, hunger, distress"; neglect "causes quietness, not suffering." This also shows up in bird pacing being tied "exclusively to aviary age."
- Slow change that preserves an "alive over weeks" illusion. The plan uses a server tick, a "low-pass filter over presence and interaction signals," "three weeks of regular presence," and drift calibration to avoid changes happening too fast.
- Server-authored continuity over client authority. The client "never writes personality state directly," the server "authors all deltas," and canonical state is the "absolute source of truth" so there is "no last-write-wins."
- Event-based state change. User actions become an "append-only event log"; the server consumes those events and applies them "sequentially," which is repeated in the sync model and risk mitigations.
- Naturalist product voice. The field notebook uses "naturalist observations," screen-reader narration is "naturalist prose," captions use the "naturalist voice," and "cheap" ARIA labels are called out as destroying the product's charm.
- Accessibility as part of the core experience. The plan treats screen-reader narration, reduced motion, captions, WCAG AA contrast, and keyboard navigation as V1 scope, not as optional polish.
- Procedural ambient sensory design. Client-side WebAudio, motif grammar, "micro-motion," "ambient weather," and non-looping calls all support a scene that remains alive without recorded loops or static state.
- Bounded, calm social presence. Social is "opt-in, read-only ambient visits via email invite," with "no co-presence," "no profiles," "public discovery," "leaderboards," "chat," or "mutual visits."
- Privacy-conscious instrumentation. Email is encrypted and "never used as an identifier elsewhere," and "no per-bird state or interaction history" goes to aggregate telemetry.

## Per-feature whys

### Scope

- Browser-based virtual aviary: NOT RECOVERABLE FROM PLAN
- Single horizontal scene, responsive: NOT RECOVERABLE FROM PLAN
- Single-user accounts with magic-link email sign-in: NOT RECOVERABLE FROM PLAN
- 2 starter birds: NOT RECOVERABLE FROM PLAN
- Capping at 7 birds: NOT RECOVERABLE FROM PLAN
- Bird availability based on aviary age: The plan ties pacing "exclusively to aviary age," aligning bird growth with time instead of "streaks, levels, scores, badges, achievements."
- Client-side procedural audio synthesis via WebAudio for bird calls: The audio pipeline says synthesis avoids "looping artifacts and phase-canceling in chorus."
- Server-side simulation tick: The tick advances aviary state "independently of clients" and lets the server hold canonical state.
- Multi-device sync: Clients read canonical server state so a laptop and phone "will see the exact same snapshot" with "no client-to-client sync or merge conflicts."
- Field notebook: The notebook gives auto-generated "naturalist observations" and preserves the plan's naturalist product voice.
- Return-greeting: NOT RECOVERABLE FROM PLAN
- Listen-in: The mixer "slowly raises the focused bird's volume while lowering (but not silencing) others," making focus possible without erasing the rest of the aviary.
- Offer (seed, song fragment, pool): NOT RECOVERABLE FROM PLAN
- Settle: NOT RECOVERABLE FROM PLAN
- Presence accounting requiring visibility, window focus, and pointer/key activity: The risk section says counting minimized tabs makes drift happen too fast and breaks the "alive over weeks" illusion.
- Screen-reader narration, reduced-motion mode, call captioning, WCAG AA contrast, keyboard navigation: The plan frames these as first-class accessibility surfaces and says generic ARIA labels would destroy charm for screen-reader users.
- Opt-in, read-only ambient visits via email invite: The plan keeps social interaction ambient and bounded by excluding "co-presence," profiles, public discovery, leaderboards, chat, and mutual visits.

### Architecture

- Client renders the scene and interpolates animations between state snapshots: NOT RECOVERABLE FROM PLAN
- Client captures interactions and writes to an append-only event log: The server consumes the log to compute drift and mood updates while preventing direct client writes to personality state.
- Server holds canonical state: The plan says the server is the "absolute source of truth" and authors all deltas.
- Chron-based simulation tick: The tick computes drift and mood updates so aviary state advances independently of clients.
- Render pipeline boundary: The plan prevents "last-write-wins" by ensuring the client never writes personality state directly.

### Data model

- `synthetic_uuid` as account primary key with encrypted email: Email is "never used as an identifier elsewhere," supporting the privacy boundary.
- Session tokens: NOT RECOVERABLE FROM PLAN
- Audio settings: NOT RECOVERABLE FROM PLAN
- Motion settings: The plan includes reduced-motion mode as an accessibility surface, swapping frame-by-frame animations for slow cross-fades.
- Notification settings: NOT RECOVERABLE FROM PLAN
- Visit log: NOT RECOVERABLE FROM PLAN
- Stable internal `bird_id`, `account_uuid`, name, species: NOT RECOVERABLE FROM PLAN
- Server-only personality vectors: The plan keeps exact personality vector values from users and prevents clients from writing personality state directly.
- Boldness, social warmth, vocal frequency, plumage saturation, curiosity: These traits drive visible and audible expression; vocal frequency drives motif timing, and pitch/timing modulation is tied to personality.
- Monotonic drift and traits never decaying on neglect: This supports the non-punitive rule that neglect causes quietness, not suffering.
- Mood persisting across sessions: This keeps current states such as wary, content, curious, drowsy, and alert from resetting just because the client closes.
- Append-only event log of user interactions: Sequential event application prevents merge conflicts and supports retry when event-log writes are dropped.
- Notebook entries with naturalist voice and timestamp: These entries preserve observations in the plan's naturalist voice over time.
- Visits with invite status and expiration: Active/revoked status and expiration support opt-in, revocable, read-only ambient visits.

### API surface

- `POST /auth/magic-link`: NOT RECOVERABLE FROM PLAN
- `POST /auth/verify`: NOT RECOVERABLE FROM PLAN
- `GET /aviary/snapshot`: This exposes canonical state including bird positions, moods, and call timing so clients read the server-authored snapshot.
- `POST /aviary/events`: This batches presence, listen-in, offers, and settle events into the append-only log instead of pushing absolute state.
- `GET /notebook`: NOT RECOVERABLE FROM PLAN
- `POST /social/invite`: This issues an email invite for the opt-in ambient visit model.
- `POST /social/revoke`: This makes ambient visits revocable.
- `GET /visit/:invite_id`: This gives visitors a read-only aviary snapshot, preserving "no co-presence."

### Simulation engine design

- Server-side tick every roughly minute per active aviary: The tick advances canonical state independently of open clients.
- Batched or lazy-evaluated tick on next read if dormant: The risk section calls for robust handling of dormant aviary ticks.
- Drift function as a low-pass filter over presence and interaction signals: This keeps change gradual and protects the "alive over weeks" illusion.
- Three weeks of regular presence producing visible changes: The plan uses this as the visible pacing target for slow drift.
- Mood transitions influenced by recent events, local time of day, ambient weather, and base personality vector: NOT RECOVERABLE FROM PLAN
- Call-grammar runtime with server motifs/timing and client WebAudio execution: The server ties motifs and timing to vocal frequency while the client executes procedural audio.

### Sync model

- Canonical server state: The server is the "absolute source of truth."
- Conflict prevention through event-only clients: Clients submit events rather than absolute personality state, so the server can apply events sequentially.
- Multi-device snapshot consistency: Simultaneous laptop and phone sessions show the "exact same snapshot."
- No client-to-client sync: The plan uses server state to avoid "merge conflicts."

### Frontend rendering pipeline

- Initial load with no loading spinners and birds mid-action: The plan wants the first frame to show birds already alive; if delayed, it uses a "quiet field" rather than a spinner.
- Scene composition with 3 perch zones and subtle parallax: NOT RECOVERABLE FROM PLAN
- Micro-motion for preening, scanning, leaves, and feathers independent of the tick: This keeps idle animation and ambient weather moving between server ticks.
- Reduced-motion mode using slow cross-fades: This preserves color shifts and procedural audio while removing frame-by-frame animation and ambient leaf drift.

### Audio pipeline

- WebAudio procedural synthesis: It avoids "looping artifacts and phase-canceling in chorus."
- Dynamic mixer for listen-in: It raises the focused bird while lowering, but "not silencing," others.
- WebAudio blocked/unavailable fallback to silence: The plan says to fail gracefully.
- Captions on by default when audio fails: This keeps call information available when sound is blocked or unavailable.
- No recorded audio fallbacks: NOT RECOVERABLE FROM PLAN

### Accessibility surfaces

- Screen-reader narration with slow-cadence naturalist prose: The plan rejects a "generic ARIA state-change log" and says cheap ARIA labels destroy charm.
- Procedural call captions in the naturalist voice: Captions carry the same naturalist voice used by other product surfaces.
- Full tab support for top-bar chrome and aviary scene: This supports keyboard navigation across both chrome and the scene.
- High-contrast focus outlines and user-copy contrast: The plan requires WCAG AA contrast for focus outlines and all user-copy text.

### Performance budgets and observability

- Initial JS bundle under 2MB gzipped: NOT RECOVERABLE FROM PLAN
- Time-to-first-bird under 500ms on mid-tier mobile over 4G: This protects quick first presentation of the bird scene on constrained mobile conditions.
- 60fps idle motion on a 5-year-old mid-range laptop: This protects smooth ambient motion on older hardware.
- No memory growth over a 30-minute session: This protects long, idle aviary sessions from degrading over time.
- Synthetic checks and aggregate RUM for page load, render frames, and tick latency: These observe whether load, rendering, and simulation timing are healthy.
- No per-bird state or interaction history in aggregate telemetry: This enforces the privacy boundary.
- Tick latency p99 alarm at 5s: This makes slow simulation ticks observable.

### Rollout

- V1 launch as web-only with magic-link auth and 2 starter birds per user: NOT RECOVERABLE FROM PLAN
- New bird species unlocks tied exclusively to aviary age: This keeps pacing out of gamified streaks, levels, scores, badges, and achievements.
- Instrumentation for tick latency, audio-context initialization errors, and presence-ping delivery rates: These correspond to risks around sync correctness, audio failure, and drift calibration.

### Risks

- Strict conjunction of visibility, focus, and pointer/key activity: This mitigates poor presence calculation that would make drift happen too fast.
- Diverse motif library and pitch/timing modulation tied to personality: This mitigates procedural audio sounding robotic when variation is too low.
- Dedicated server/client logic for naturalist prose narration: This mitigates accessibility regressions from cheap ARIA labels.
- Reliable retry mechanisms for the append-only event log: This mitigates dropped writes that could lose user presence time.
- Robust handling of dormant aviary ticks: This mitigates sync correctness issues when aviaries are not actively open.
