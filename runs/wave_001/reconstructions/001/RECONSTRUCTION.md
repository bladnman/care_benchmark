# Pocket Aviary Reconstruction

## System-level intent

1. **A tightly bounded V1.** The plan opens by naming "Scope and Boundaries" and repeatedly draws hard lines: "In-Scope for V1" includes a small aviary, core interactions, accounts, naturalist surfaces, read-only social, accessibility, and performance, while "Out-of-Scope" excludes native applications, gamification, Tamagotchi-style custodial mechanics, and social networking.

2. **A soft, non-gamified, non-custodial aviary.** The plan carries an anti-game philosophy through "No streaks, achievements, XP, levels, scores, visit calendars, or user progress metrics" and through "Birds do not die, decay, get sick, or show distress on neglect." The "Settle" interaction is described as an "opt-in soft exit gesture," reinforcing that the product should not punish absence or push progress.

3. **Bird-centered naturalist voice.** The plan explicitly calls for "Naturalist-style product copy" that is "lowercase, present-tense, observational, bird-centered." This shows up again in the "Field Notebook" as a "sparse, auto-generated, read-only diary of naturalist observations" and in the lowercase generated example: "thursday -- wren perched on the high branch, calling softly."

4. **Presence should shape the aviary slowly.** The plan treats attention as "real-time passive tracking" and converts it into an "Effective Attention Window." The drift formula is calibrated to "about 1% vector movement per 3 hours of continuous focus," and deltas are clamped so "negative updates are impossible." This suggests slow, monotonic change rather than fast reward loops.

5. **Server-ticked canonical state over client-owned state.** The account model promises "Multi-device synchronization reading from a single server-ticked canonical state." The sync section names a "Single Writer Pattern," says "Only the server-side simulation tick modifies the state of the birds," and has clients submit "behavioral actions rather than state values" to prevent "race conditions and overwrites."

6. **Privacy and PII protection by design.** The plan says "Synthetic account IDs (UUIDv4) to prevent PII leakage," stores `encrypted_email`, hashes guest email addresses, and names "PII Data Leakage" as a high-impact risk. Its mitigation is to "Restrict email addresses to the main account table" and "Use UUIDv4 values across all system processes and metrics trackers."

7. **Social access is bounded and read-only.** The social surface is "Read-only visitor links sent via email" with "No co-presence or live interaction." The guest state endpoint is "Blocked from write endpoints," and broader social networking is excluded: no profiles, discovery feeds, mutual follows, comments, or shared/collaborative aviaries.

8. **Accessibility surfaces translate the same state into other forms.** "Screen-Reader Narration" gives "Dynamic prose descriptions of the active state." "Reduced-Motion Mode" swaps visual transition mechanics for cross-fades. "Call Captions" map synthesis triggers to text overlays describing call qualities. These are presented as parallel renderings of aviary state, not separate products.

9. **Ambient media should be procedural, spatial, and responsive.** The rendering plan uses a responsive SVG scene with background, midground, and foreground groups. The audio plan uses WebAudio for "real-time client-side procedural sound generation," maps bird position to stereo pan, and lowers back-perch gain and filter frequency "to simulate depth."

10. **Performance budgets are product requirements.** The plan states aggressive targets in scope and repeats them under "Performance Budgets & Observability": initial bundle "<2MB," "Time-to-First-Bird: <500ms," and "60fps target on 5-year-old laptops." The validation plan tests these with Lighthouse and memory checks.

## Per-feature whys

### Scope and Boundaries

- **Aviary Capacity**: NOT RECOVERABLE FROM PLAN

- **Presence**: The plan describes presence as "Real-time passive tracking of user attention." In the simulation section, `presence_ping` events become the "Effective Attention Window," which drives slow trait drift.

- **Listen-in**: The plan says listen-in is for "Focusing a specific bird to adjust the audio mix." In simulation it applies extra movement to the target bird's `social_warmth` and `vocal_frequency`; in audio it ramps other birds down and the target bird to full volume.

- **Offers**: The plan describes offers as "Cooldown-limited interaction items" and says successful offer completions apply movement to `curiosity` and `boldness`. The plan does not explain why these interactions are seed, song fragment, and still pool.

- **seed**: NOT RECOVERABLE FROM PLAN

- **song fragment**: NOT RECOVERABLE FROM PLAN

- **still pool**: NOT RECOVERABLE FROM PLAN

- **Settle**: The plan calls settle "an opt-in soft exit gesture that shifts lighting and settles the aviary."

- **Email-based magic-link authentication**: The plan articulates a one-time account access flow with "15-minute token expiration" and rate limiting, but it does not explain why magic links were chosen.

- **Multi-device synchronization**: The rationale is to read from "a single server-ticked canonical state" so multiple devices render the same aviary state and avoid collisions.

- **Synthetic account IDs**: The plan states the rationale directly: "to prevent PII leakage."

- **Field Notebook**: The plan frames this as "a sparse, auto-generated, read-only diary of naturalist observations." The generation section limits density to "1 entry every 2-3 days for active users to prevent log fatigue."

- **Naturalist-style product copy**: The plan gives the voice rationale as "lowercase, present-tense, observational, bird-centered."

- **Read-only visitor links sent via email**: The plan uses read-only visitor links to allow social viewing while preserving the boundary of "No co-presence or live interaction."

- **Screen-Reader Narration**: The plan says this provides "Dynamic prose descriptions of the active state."

- **Reduced-Motion Mode**: The plan says this replaces "frame-by-frame animations" with "Visual transition cross-fades."

- **Call Captions**: The plan says these are "Generative text overlays describing call qualities" and later maps sound synthesis triggers to text overlays.

- **Aggressive performance optimization**: The plan gives explicit product constraints: initial bundle "<2MB," first-bird-render "<500ms," and "60fps on 5-year-old laptops."

### Architecture & Tech Stack

- **Web Browser Client / API Gateway / CDN Edge Cache / PostgreSQL / Redis / Simulation Tick Worker**: The architecture supports event submission, state fetching, durable storage, event buffering, and worker-owned state mutation, but the plan does not give a separate rationale for this exact topology.

- **Vanilla HTML5, CSS3 Custom Properties, ES6+ TypeScript, Vite**: NOT RECOVERABLE FROM PLAN

- **WebAudio API**: The plan says this is for "real-time client-side procedural sound generation."

- **Node.js with TypeScript and Express**: NOT RECOVERABLE FROM PLAN

- **Background processing queue running on a Node-based worker task runner**: The plan uses this for the "Simulation Tick Engine," where background ticks process events and mutate state.

- **PostgreSQL**: The plan states this is "for durable data storage."

- **Redis**: The plan states this is "for API session tokens, rate limiting, and buffering event queues."

### Data Model

- **accounts**: The `encrypted_email` field is labeled "PII protection," and the account ID defaults to UUID. The `deleted_at` field is labeled "Soft delete for 30 days."

- **sessions**: NOT RECOVERABLE FROM PLAN

- **birds**: The plan stores a "Hidden Personality Vector" and short-term state. The later simulation and rendering sections use these fields for trait drift, mood transitions, perch zones, and plumage saturation.

- **interaction_events**: The plan labels this an "Append-Only Interaction Event Log." The sync model explains the rationale: clients submit "behavioral actions rather than state values," preventing "race conditions and overwrites."

- **notebook_entries**: The table stores generated field notebook text so the offline naturalist grammar can write entries directly.

- **visit_invitations**: The table supports visitor links with guest email hashes, tokens, status, expiration, and revocation, matching the read-only social model.

- **visit_log**: NOT RECOVERABLE FROM PLAN

### API Surface

- **POST /api/auth/request-link**: The endpoint verifies or creates an account, sends a one-time link with a "15-minute validity window," and rate-limits requests to "3 requests per 15 minutes."

- **POST /api/auth/verify-link**: The endpoint turns a token into a `session_token` and `user_uuid` and stores the token in an "HTTPOnly, Secure, SameSite=Strict" cookie.

- **POST /api/auth/logout**: The endpoint revokes the current session token.

- **GET /api/aviary/state**: The endpoint returns the current aviary snapshot so the client can render birds, mood, perch zone, time of day, weather, and plumage saturation.

- **POST /api/aviary/events**: The endpoint validates the session token and appends events to the backend log, matching the append-only client update model.

- **POST /api/social/invite**: The endpoint generates a custom visit link for read-only visitor access.

- **POST /api/social/revoke**: The endpoint supports revoking an invitation.

- **GET /api/guest/state/:token**: The endpoint retrieves a read-only host aviary snapshot, is rate-limited, and is "Blocked from write endpoints."

### Simulation Engine Design

- **Server-Side Simulation Tick**: The tick processes accounts with client activity in the last 15 minutes, fetches events, computes attention, mutates state, and writes the final payload. The specific 60-second cadence is NOT RECOVERABLE FROM PLAN.

- **Effective Attention Window**: Each `presence_ping` represents "a maximum 30-second block of validated active presence," allowing passive attention to become bounded simulation input.

- **Drift Factor**: The formula moves traits slowly toward their upper bound, with calibration translating to "about 1% vector movement per 3 hours of continuous focus."

- **Monotonic check**: The plan says deltas are clamped to `>= 0`, and the validation section asserts that "negative updates are impossible."

- **listen_in_start modifiers**: The plan applies extra delta to the target bird's `social_warmth` and `vocal_frequency`, connecting focused listening to social and vocal traits.

- **offer completions modifiers**: The plan applies movement to `curiosity` "if accepted" and `boldness`, connecting successful offers to exploratory and confidence traits.

- **Mood Engine Execution**: The engine uses timezone and weather for baseline probability distributions, Markov state transitions for moods, boldness to scale down `wary`, and social warmth to increase `curious` or `content` when neighboring birds vocalize.

- **Dynamic Field Notebook Generation**: The script looks for "landmark transition" moments such as first greeting of the week, weather change, or perch-zone shift, then assembles a lowercase naturalist entry. The density limit exists "to prevent log fatigue."

### Multi-Device Sync Model

- **Single Writer Pattern**: The plan states the rationale directly: "Only the server-side simulation tick modifies the state of the birds."

- **Append-Only Client Updates**: The plan says clients submit "behavioral actions rather than state values" to prevent "race conditions and overwrites."

- **Device Conflict Resolution**: The plan makes the client "a reactive rendering engine," fetches snapshots on focus, visible transitions, or every 30 seconds, and processes simultaneous device pings sequentially so "the server aggregates the attention without collision."

### Frontend Rendering Pipeline

- **Scene Composition**: The plan uses a responsive SVG overlay with Background, Midground, and Foreground nested groups. The rationale for this exact composition is NOT RECOVERABLE FROM PLAN.

- **Idle Micro-motion**: The plan describes separate bird sub-shapes and randomized CSS animation delays, but the rationale is NOT RECOVERABLE FROM PLAN.

- **Motion Interpolation**: The plan says that when perch position changes, the client avoids immediate repositioning and instead interpolates along a Bezier path over 800ms.

- **Reduced-Motion Mode**: The plan disables continuous keyframe animations and replaces coordinate translation paths with opacity cross-fades.

### Audio Pipeline

- **Procedural Synthesis Engine**: The plan says calls are "mathematically synthesized rather than loaded from external audio files" using native WebAudio nodes.

- **Pitch Sweep Logic**: NOT RECOVERABLE FROM PLAN

- **Spatial Panning & Mixing**: The plan maps bird horizontal center to stereo pan and uses low-pass filtering plus lower gain for back perch birds "to simulate depth."

- **Listen-in Mix Decay**: The plan ramps other active birds down and the target bird up, then smoothly returns all gains to ambient mix levels on release.

### Accessibility Surfaces

- **Narration Engine**: The plan uses an `aria-live="polite"` element and translates spatial layouts and mood updates into descriptive prose. The cadence avoids over-updating at idle while triggering immediately on interactions.

- **Call Captioning**: The plan maps active sound synthesis triggers to text overlays near the bird and enforces contrast with `rgba(0, 0, 0, 0.8)` and white text for WCAG AA.

### Performance Budgets & Observability

- **Initial JS Bundle**: The plan targets "<2MB" and says this is achieved via "aggressive tree-shaking and dynamic route importing for Settings/Notebook code."

- **Time-to-First-Bird**: The plan targets "<500ms," starts rendering with fallback inline styling, and inlines state snapshots into the server's initial HTML "to avoid additional fetch requests."

- **Runtime Performance**: The plan targets "60fps" and uses CI memory checks to prevent growth over a "30-minute running session."

### Rollout & Validation Plan

- **Simulation unit testing**: The plan verifies monotonic vector updates and asserts that negative updates are impossible.

- **Integration and browser tests**: The plan uses Headless Chrome and Lighthouse to verify Time-to-First-Bird and bundle targets.

- **WebAudio fallback regression tests**: The plan disables the Audio Context and verifies that call caption triggers still work.

- **Manual layout verification**: The plan checks small mobile and large screen devices to validate that "no birds are cropped out of the viewport."

### Risks and Mitigations

- **Drift Calibration Imbalance**: The plan mitigates this with simulated user interaction scripts spanning "60 virtual days" to validate that drift remains slow and within target visual increments.

- **Sync Race Conditions**: The plan mitigates this by maintaining the single-writer database pattern and blocking clients from updating coordinates directly.

- **Robotic Sound Generation**: The plan mitigates this with "micro-timing jitter" and frequency variation in the WebAudio synthesis code.

- **PII Data Leakage**: The plan mitigates this by restricting email addresses to the main account table and using UUIDv4 values across system processes and metrics trackers.
