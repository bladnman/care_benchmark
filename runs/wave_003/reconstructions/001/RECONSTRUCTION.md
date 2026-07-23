## System-level intent

- **Feels alive, not robotic**: The plan names this as a core design principle and ties it to "server-side simulation continuous ticking," "instant snapshot hydration with motion mid-progress," "procedural audio synthesis via WebAudio," and "mood-driven micro-motion." It shows up again in the 60-second server simulation tick, persisted mood states, first-frame birds "mid-action," and the absence of loading spinners.
- **Notice, never announce**: The plan defines this through "zero return toasts, zero streak counters, zero push notifications, zero gamification dialogs." Greetings should be "performed purely by bird behavior" such as a "glance, step forward, quiet call." This also appears in the exclusions of gamification and in the quiet social model.
- **Charm comes from specificity**: The plan grounds charm in "naturalist, present-tense, lowercase field-notebook observations" and screen-reader narration "over generic state logs." It appears in the field notebook examples, procedural call captions, and shared naturalist narration surfaces.
- **Restraint over richness**: The plan repeats restraint through a "capped single-screen scene," "max 7 birds," "3 perches," "minimal top-bar chrome," cursor-idle fading, and "web-only architecture." The non-goals reinforce the same restraint by excluding native apps, gamification, Tamagotchi mechanics, and social network features.
- **Naturalist voice vs. Matter-of-fact system voice**: The plan separates "naturalist prose across product/notebook/narration surfaces" from "matter-of-fact, clear language across auth, error, sync, and settings surfaces." This voice split appears in authentication failure language, notebook prose, and screen-reader narration.
- **Server as sole canonical writer**: The plan repeatedly prevents client authority over personality state: "server is the sole canonical writer," "single canonical writer," "No Last-Write-Wins," and clients submit "un-opinionated interaction events." This shows up in scope, sync, the simulation worker, the append-only event log, and the LWW risk mitigation.
- **Gentle, monotonic change over days and weeks**: The plan frames personality drift as slow and additive: "personality vector slowly drifts," "monotonic low-pass personality drift," "Traits never decrease due to neglect or absence," and "birds never die, starve, decay, or exhibit distress." The one-week and three-week calibration targets keep change neither too fast nor static.
- **Privacy and telemetry boundary**: The plan uses "synthetic UUID" account keys, encrypted email storage, email hashes "for auth lookups only," and "strict data exclusion" for personality vectors, interaction events, notebook prose, and presence logs. Social visit data is read-only, revocable, masked in logs, and has no visitor drift impact.
- **Accessibility with naturalist narration and full functionality**: Accessibility features keep the same product voice and mechanics: "naturalist screen-reader narration," reduced-motion cross-fades that preserve "full audio, drift, and notebook functionality," procedural call captions, keyboard navigation, and WCAG AA contrast.

## Per-feature whys

### Executive Summary & Design System Alignment

- **Modern, browser-native virtual aviary**: The plan's rationale is "web-only architecture" and a browser client using HTML5 Canvas/SVG and WebAudio.
- **Two to seven animated birds in a single horizontal scene**: The upper bound supports "Restraint over richness," the "capped single-screen scene," the 3-perch scene, and audio distinctness under the 7-bird cap.
- **Personality vector drift over days and weeks**: The plan's rationale is long-horizon aliveness: personality "slowly drifts in response to measured user presence and quiet interactions."

### 1. Scope & System Boundaries

- **Two automatically assigned starter birds**: NOT RECOVERABLE FROM PLAN
- **Cap of 7 birds total**: The plan ties the cap to "Restraint over richness," a "capped single-screen scene," and preserving "distinct audible call signatures."
- **Progressive unlocks based strictly on aviary age**: NOT RECOVERABLE FROM PLAN
- **Email magic-link authentication with 15-minute expiration**: NOT RECOVERABLE FROM PLAN
- **Synthetic UUID internal mapping**: The plan ties this to privacy and persistence boundaries: tables key off synthetic account UUIDs, email is encrypted, and email hash is "for auth lookups only."
- **Server-side canonical simulation engine**: The rationale is continuous aliveness and canonical state: a server-side 1-minute tick drives drift, mood state machines, and field notebook generation while the server remains the sole writer of personality state.
- **Client snapshot consumption plus append-only event log submission**: The rationale is multi-device conflict prevention: clients never write absolute trait values, and the server consumes un-opinionated events.
- **Measured presence accounting**: The rationale is to feed drift from qualified presence only, using the conjunction of visible tab, focused window, and recent pointer/key activity.
- **Listen-in audio focus**: The plan's rationale is focused listening without fully muting the aviary: selected birds gain volume while non-focused birds remain at an ambient background level.
- **Seed, song, and pool offers**: The plan connects offers to interaction events and mood updates through "interaction weights" and "recent offer responses."
- **Offer cooldowns**: NOT RECOVERABLE FROM PLAN
- **Soft settle session-end gesture**: NOT RECOVERABLE FROM PLAN
- **Read-only field notebook**: The rationale is "Charm comes from specificity," using naturalist observations instead of generic state logs.
- **Host-initiated read-only ambient visit invitations**: The rationale is optional, quiet social access: "default off," "fully revocable," "no co-presence," and "no visitor drift impact."
- **Naturalist screen-reader narration**: The rationale is accessibility in the same naturalist, present-tense product voice.
- **Reduced-motion mode**: The rationale is accessibility while preserving "full audio, drift, and notebook functionality."
- **Procedural call captioning**: The rationale is an accessible fallback and companion to generated bird calls.
- **Keyboard navigation and WCAG AA contrast**: The rationale is full accessibility across controls and bird navigation.
- **Performance budgets**: The rationale is to maintain instant and continuous aviary behavior: quick first render, 60fps motion, and no memory growth.

### 1.2 Non-Goals & Explicit Exclusions

- **No native apps**: The rationale is "web-only" and "browser-native" scope.
- **No gamification**: The rationale is "Notice, never announce," with absolute exclusion of streaks, visit dots, levels, scores, badges, adoption counters, XP, and ranking.
- **No Tamagotchi mechanics**: The rationale is that birds "never die, starve, decay, or exhibit distress," and neglect produces quiet ambient behavior rather than negative personality drift.
- **No social network features**: The rationale is keeping social optional and quiet, with no public profiles, comments, chat, co-presence cursors, discovery feeds, or leaderboards.

### 2. System Architecture & Technical Topology

- **Browser Client**: The rationale is browser-native rendering, WebAudio synthesis, local presence evaluation, snapshot polling, interpolation, and append-only event emission.
- **CDN / Edge Tier**: The rationale is static asset delivery under the <2MB budget and initial snapshot bootstrap.
- **Aviary API Gateways**: NOT RECOVERABLE FROM PLAN
- **Simulation Worker Service**: The rationale is a background 1-minute tick that ingests events, runs low-pass drift, advances mood, and writes updated canonical state.
- **Event Store & Primary DB**: The rationale is storing the append-only event log, canonical aviary and bird vectors, field notebook, visit log, and synthetic UUID account index.
- **Relational PostgreSQL data persistence**: The rationale is canonical state and event storage keyed off synthetic account UUIDs.

### 3. Data Model & Schema Specifications

- **Accounts table with encrypted email and email hash**: The rationale is privacy and auth lookup separation: email is encrypted at rest, and the hash is "for auth lookups only."
- **Aviaries table with one aviary per account at v1**: NOT RECOVERABLE FROM PLAN
- **Birds table with species, name, and slot_index 0 to 6**: The rationale is the capped 7-bird scene and stable bird identity within the aviary.
- **Hidden normalized personality vectors**: The rationale is that user-facing surfaces should show naturalist observations rather than generic state logs, scores, or gamified counters.
- **Bird mood states**: The rationale is fast-timescale daily/session state that persists across sessions and does not reset on tab open.
- **Append-only interaction event log**: The rationale is conflict prevention and drift input without client-written absolute trait values.
- **Field notebook entries**: The rationale is naturalist, present-tense observations generated from the simulation.
- **Social visit invitations**: The rationale is quiet, revocable, host-initiated read-only visits.
- **Visit audit log**: NOT RECOVERABLE FROM PLAN

### 4. API Surface & Contract Specifications

- **Magic-link request and verify endpoints**: NOT RECOVERABLE FROM PLAN
- **Session revocation endpoint**: NOT RECOVERABLE FROM PLAN
- **Account JSON export endpoint**: NOT RECOVERABLE FROM PLAN
- **30-day soft deletion endpoint**: NOT RECOVERABLE FROM PLAN
- **Aviary snapshot endpoint**: The rationale is fetching the current canonical state for client rendering and hydration.
- **Batch interaction events endpoint**: The rationale is append-only submission of presence, listen-in, offer, and settle events to feed server-side simulation.
- **Notebook fetch endpoint**: The rationale is read access to naturalist notebook observations.
- **Visit invite, revoke, and read-only snapshot endpoints**: The rationale is optional quiet social access that is host-initiated, revocable, and read-only.

### 5. Server-Side Simulation Engine & Drift Mechanics

- **Asynchronous 60-second simulation tick**: The rationale is continuous server-side aliveness and canonical updates.
- **Event ingestion during each tick**: The rationale is consuming interaction events since the last tick before calculating drift and mood changes.
- **Presence and interaction weights**: The rationale is converting qualified presence seconds and interactions into simulation inputs.
- **Monotonic low-pass personality drift**: The rationale is slow additive change where Delta T is non-negative and traits "never decrease due to neglect or absence."
- **One-week and three-week calibration targets**: The rationale is making drift instrument-detectable after regular presence and visibly perceptible later, while avoiding drift that is too fast or static.
- **Mood state machine with time-of-day and weather modulator**: The rationale is daily/session behavior influenced by timezone hour, recent offers, ambient weather, and personality thresholds.
- **Persisted mood across sessions**: The rationale is continuity; mood "does not reset on tab open."
- **Field notebook generation**: The rationale is naturalist, present-tense, lowercase prose generated from sparse rule triggers.
- **Once every 2-3 days notebook cadence**: NOT RECOVERABLE FROM PLAN

### 6. Multi-Device Sync & Conflict Prevention

- **Single canonical writer**: The rationale is preventing competing writers; only the server simulation tick writes personality vectors and moods.
- **No Last-Write-Wins**: The rationale is avoiding LWW data loss by having clients submit un-opinionated events rather than absolute trait values.
- **Snapshot refresh on visibility restore, long frame gaps, or keepalives**: The rationale is state hydration from canonical snapshots.
- **1,500ms client-side interpolation**: The rationale is smooth motion between snapshots using easing curves.

### 7. Frontend Rendering Pipeline & Scene Design

- **Fixed single horizontal scene with no horizontal scrolling or panning**: The rationale is a restrained single-screen aviary that fits mobile to desktop.
- **Three perch zones**: The rationale is visible depth planes for behavior-driven placement.
- **Personality/mood-driven bird placement**: The rationale is that bird position expresses boldness and mood, such as high boldness on the front perch and wary mood on the back perch.
- **Disabled user placement**: NOT RECOVERABLE FROM PLAN
- **Top bar icons for settings/account, accessibility, notebook, and offers**: NOT RECOVERABLE FROM PLAN
- **Top bar fading to 5% opacity after stillness**: The rationale is minimal top-bar chrome with cursor-idle fading.
- **First frame birds mid-action**: The rationale is "Feels alive, not robotic" and instant scene initialization.
- **Quiet soft sky loading fallback with zero loading spinners**: The rationale is preserving the quiet, non-robotic first impression.
- **Reduced-motion cross-fade rendering**: The rationale is accessibility with slow pose transitions while disabling particles and preserving audio, drift, and notebook functionality.

### 8. WebAudio Procedural Synthesis & Audio Pipeline

- **Runtime motif synthesis with WebAudio FM synthesis and filtered noise**: The rationale is procedural bird calls without downloaded audio samples.
- **Species motif library**: NOT RECOVERABLE FROM PLAN
- **Chorus pitch and timing micro-randomization**: The rationale is preventing phase cancellation between calling birds.
- **7-bird audio cap**: The rationale is preserving distinct audible call signatures.
- **Listen-in gain ramp and ambient reduction**: The rationale is focusing one bird's call while keeping other birds audible and "never fully muted."
- **Silent audio fallback with automatic call captions**: The rationale is graceful behavior when WebAudio is unsupported or permission is denied.

### 9. Accessibility Implementation

- **Dedicated polite ARIA live region**: The rationale is naturalist screen-reader narration updated without being overly disruptive.
- **Naturalist summaries every 30-60 seconds or on user events**: The rationale is present-tense narration in the same naturalist voice as the field notebook.
- **Procedural call captions near calling birds**: The rationale is naturalist text for bird calls, driven by the active WebAudio motif generator parameters.
- **Keyboard focus and navigation**: The rationale is full keyboard accessibility for top bar controls, bird navigation, listen-in, and disengage.
- **High-contrast visual focus ring**: The rationale is WCAG AA contrast against day/night palettes.

### 10. Performance Budgets & Observability

- **Initial JS bundle under 2MB gzipped**: The rationale is static asset delivery within the hard performance budget.
- **Time-to-first-bird under 500ms**: The rationale is quick first visible bird render on 4G mid-tier mobile.
- **60fps continuous idle motion**: The rationale is smooth runtime performance on 5-year-old laptop hardware.
- **Zero memory growth over 30 minutes**: The rationale is avoiding leak growth, verified with CI Chrome DevTools protocol tests.
- **Allowed operational metrics**: The rationale is operational observability for request rates, tick latency p99, WebAudio errors, and anonymized page load histograms.
- **Strict telemetry data exclusion**: The rationale is the privacy boundary excluding personality vectors, per-account events, notebook prose, and presence logs.

### 11. Rollout & Risk Management

- **Phase A internal canary**: The rationale is synthetic tick validation, drift low-pass filter verification, and WebAudio cross-browser testing.
- **Phase B beta release**: The rationale is limiting rollout to a 10% account cohort while enabling starter birds and age-based unlocks.
- **Phase C general availability**: The rationale is full rollout of multi-device sync and social visit invitations after earlier phases.
- **Drift calibration CI harness**: The rationale is mitigating drift that occurs too fast or feels static by asserting vector bounds across 7-day and 21-day simulated tick streams.
- **Strict append-only event log enforcement**: The rationale is mitigating critical multi-device LWW data loss.
- **Shared screen-reader and field notebook template engine**: The rationale is mitigating robotic screen-reader narration.
- **WebAudio autoplay fallback**: The rationale is mitigating autoplay blocking with a muted state and a user gesture listener that restores audio context.
