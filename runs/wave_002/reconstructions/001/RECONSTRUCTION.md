## System-level intent

- The aviary should feel as though it continues without the viewer. This shows up in the server-side simulation tick, the first frame where birds are "already in motion", the instruction to render "as if it's been rendering all along", and the loading-state risk that warns against breaking "the central conceit that the aviary has been continuing without the viewer."

- The product is observational, not custodial. The plan excludes "Tamagotchi mechanics", says there is "no negative drift on neglect", makes personality drift "monotonic toward expressive", and names the affective success criterion that "the relationship is observational, not custodial."

- The user should feel accompanied, not entertained. This is carried by "No gamification", by the refusal of achievements, streaks, levels, scores, badges, and by the affective criterion that the "User feels accompanied, not entertained."

- The product should notice, never announce. The appendix says "no 'Welcome back!' toast, no level-up confetti, no badge popups"; top-bar chrome fades after cursor stillness; visit invitations avoid push, email, or in-product notification by default.

- Charm comes from specificity. The plan names "naturalist prose", "specific to bird and moment", per-bird motifs, procedural call captioning that matches what was actually played, and field notebook entries with context like time of day, weather, and notable events.

- Naturalist voice belongs on aviary surfaces, while system surfaces stay matter-of-fact. The plan says "Naturalist voice everywhere" for product surface, field notebook, screen-reader narration, and call captions, but uses "Matter-of-fact voice for system surfaces" for sign-in, errors, sync conflicts, and accessibility settings.

- Accessibility is a first-class product surface, not a checklist. The plan says reduced-motion mode is "not a fallback" and "its own designed surface", screen-reader narration is "naturalist prose, not state lists", and the accessibility regression risk warns against "teaching users that access to the product's quality is rationed by sensory ability."

- Restraint over richness is a design principle. This appears in "2-7 birds, one screen, calm color palette, no UI chrome inside aviary", "Single horizontal scene", "No UI inside the aviary", and out-of-scope boundaries around customization, public discovery, leaderboards, and social network surfaces.

- Canonical server state protects continuity and correctness. The plan says the "Server is the only writer" of personality vectors and moods, clients are "read-only", all clients read "one canonical record", and there is "no merging, no last-write-wins."

- Privacy boundaries shape identifiers, telemetry, and social features. The account uses a "synthetic UUID, never email"; email is "encrypted"; Real User Monitoring is "aggregate-only"; there is "No per-bird state in telemetry"; visits are opt-in, read-only, and private to named invitees.

## Per-feature whys

### Scope

- Single-user accounts: The plan frames the aviary as private to one host account, rejects "shared aviaries, household accounts, or multi-user accounts", and keeps the product away from social network surfaces.

- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account: The reason is multi-device coherence: "all clients read from one canonical record" so laptop and phone show "the same aviary, same moods, same drift", with no aviary switching or merge surface.

- Two starter birds: NOT RECOVERABLE FROM PLAN

- Max 7 birds: The plan says this cap is "empirical, not arbitrary" and is the "audio recognizability ceiling"; it may increase only if future audio-mix work raises that ceiling.

- Web-only v1: The plan excludes native mobile apps and says native may be revisited later but v1 is "not designed for native-client constraints."

- No gamification: The plan says no achievements, streaks, levels, scores, or badges, and adds that the cumulative effect of "just one" is "total failure." This supports the affective goal that the user feels "accompanied, not entertained."

- No Tamagotchi mechanics: The plan rejects death, hunger, distress, and happiness decay. Its rationale is carried by "no negative drift on neglect", "avoids Tamagotchi model", and the desired relationship as "observational, not custodial."

- No social network surfaces, public discovery, leaderboards, or rankings: The plan keeps the aviary private to "host and named invitees only", removes public feeds and comments, and says there is "no ranking of any kind."

- No push notifications, emails, or pings about the aviary: The plan says the "aviary lives where user visits it", matching the "notice, never announce" philosophy.

- No payments, billing, or premium tiers: NOT RECOVERABLE FROM PLAN

- No customizable scenes, bird selection, or species catalog: NOT RECOVERABLE FROM PLAN

### Architecture and data

- Single monolithic service for v1: NOT RECOVERABLE FROM PLAN

- PostgreSQL as the database choice: NOT RECOVERABLE FROM PLAN

- Redis as the cache choice: NOT RECOVERABLE FROM PLAN

- Append-only event log per account: The plan uses it for interaction events consumed by the tick, preserves "event log order", and prevents clients from writing personality state directly.

- Server/client split: Server responsibilities hold account management, canonical state, tick, drift, mood, notebook, visits, and telemetry; client responsibilities are rendering, audio synthesis, interpolation, presence detection, event submission, keyboard navigation, narration, reduced motion, and captions. The reason is that the server owns truth while the client renders the experience smoothly.

- No client-side simulation: The plan states that the client "never computes drift, mood transitions, or personality updates" and "only renders what the server sends", protecting the server-authored state model.

- Synthetic account UUID and encrypted email: The plan says account IDs are "never email" and email is encrypted, which supports the privacy boundary.

- Soft-delete and restore: NOT RECOVERABLE FROM PLAN

- Account settings for visit notifications, captions, reduced motion, and audio muted: The rationale is user control over accessibility and quiet social behavior: captions and reduced motion are accessibility surfaces, audio may be muted, and visit notifications are off by default.

- JSON account export: NOT RECOVERABLE FROM PLAN

### Simulation engine

- Server-side simulation tick: The reason is to make "the aviary continues without the viewer" actually true while applying drift, mood transitions, idle animation state, and notebook generation to canonical state.

- Tick cadence of about once per minute: The plan treats this as a calibrated build parameter and sets a latency alarm because the tick is "supposed to take much less" than five seconds.

- Presence accounting: Presence is the dominant drift input and must require visibility, focus, and pointer/key activity. The plan warns that treating "tab open = presence" would corrupt drift across the user base, while "watching birds without moving is the actual product."

- Personality drift: The rationale is slow, expressive change over weeks. Traits move up on positive presence, never down on neglect, no single session shifts a trait visibly, and drift should be measurable after about one week and visible after about three weeks.

- Listen-in as a drift input: The plan calls listen-in "strong" because it means "focusing a bird", and it affects social warmth and vocal frequency.

- Offers as a drift input: Offers are "small"; accepting offers drifts curiosity, and offering near birds drifts boldness.

- Settle as a drift input: Settle is "quiet" and "ends presence window cleanly" with "no drift direction."

- Mood system: The plan distinguishes mood from personality with recent interactions, time of day, ambient events, and personality vector as inputs; mood supports the success criterion that birds have "mood (not states)."

- Mood persistence: Mood at session-end carries to session-start so birds "don't reset to neutral on tab open."

- Bird-to-bird interaction: Calls can prompt responses, wary mood can spread, and chorus events happen when birds call in the same window; the rationale is a more alive aviary with relationships among birds.

- Field notebook: The plan uses low-cadence "naturalist prose" with notable events, keeping the same field-notebook voice and making observations specific to bird and moment.

### Frontend rendering

- Single horizontal scene: The plan keeps the aviary on "one screen" with no panning, scrolling, or zooming, supporting restraint and making all birds present at once.

- Three perch zones: The plan says front, middle, and back are a "signal for user, not user-controlled."

- Responsive viewport handling and preserved aspect ratio: The reason is that "all birds always visible, never cropped or offscreen."

- First frame with birds already in motion: This protects the conceit that the aviary has been continuing; there is no "wake up" animation.

- Idle micro-motion: Preening, scanning, head-tilt, and body-shuffle are "mood-shaped", helping the aviary feel alive rather than robotic.

- Ambient leaf and feather drift with parallax: The plan treats this as client-side idle-cadence motion that adds life to the place while keeping it ambient.

- Day/night cycle: The plan uses local time for gradual palette shifts and mood modulation.

- Rare ambient rain and wind: The plan gives these short-lived mood effects, including rain dampening vocal frequency.

- Client stops rendering when the tab is hidden: The plan gives the reason as "battery saving", while simulation continues server-side.

- Reduced-motion rendering: The plan says this is not "animations off" and not a fallback, but "a different rendering of the same aviary" with cross-fades and its "own charm."

- Top-bar chrome: The plan keeps icons in the top bar only, keeps "birds and place only" inside the aviary, fades chrome after stillness, and supports restraint over richness.

- Quiet loading state: The reason is to avoid showing a loading or wake-up moment that would break the continuing-aviary conceit; if needed, it should be a "quiet field" with soft sky color and faint motion cues.

### Audio

- Procedural call synthesis via WebAudio: The plan gives "bundle budget + chorus mechanics + no canned audio" as the reason for WebAudio and no recorded calls.

- Per-bird motif library: Species-specific motifs with personality-shaped timing and pitch support unique per-bird calls, not "3 variants in rotation."

- Call grammar runtime: Procedural call text is generated from the motif at runtime so captions can match what was actually played.

- Chorus mixing: The plan requires real-time mixing of procedural calls, "not stacked loops", to avoid canned or phase-canceling artifacts and to make a real chorus.

- Listen-in mix: Focusing one bird gradually raises its mix level and drops others without muting them; the reason is that it "feels like listening, not switching channels."

- WebAudio fallback: If WebAudio is unavailable, the plan prefers graceful silence with captions on by default because "Silence with captions is better fallback than canned audio" and recorded fallback would collapse the bundle budget.

- Audio memory budget: Reusing buffers and avoiding unreleased per-call allocation supports "No memory growth over 30 minutes."

### Accessibility surfaces

- Screen-reader narration: The rationale is access to the same naturalist quality as the rest of the product. It is "not state lists, not ARIA-label automation", uses a slow cadence, and keeps "voice continuity."

- Reduced-motion mode: The plan repeats that this is a designed surface with its own charm; birds still drift, mood still changes, and "the aviary is still the aviary."

- Call captioning: Captions are short naturalist prose, placed near the calling bird, and procedurally generated so they match the actual call.

- Keyboard navigation: The plan makes the top bar, birds, listen-in, offer, settle, and exit flows keyboard-accessible so accessibility surfaces are complete.

- Focus indicators: The reason is visibility against the aviary background with a "soft, high-contrast outline."

- WCAG AA contrast: The plan applies it to all user-copy text, especially top bar labels, settings, account surfaces, error surfaces, captions, and narration when visually displayed.

### Visits

- Visit-invitation feature: The plan makes visits read-only, opt-in, and per-invite so the product has named visitors without becoming a social network.

- Visit revocation and visit list/log: The plan frames visit management as host-owned, with invitation list and visit log visible to the host.

- Read-only ambient visit view: The plan rejects chat, avatars, comments, public discovery, leaderboards, and "show-off" mode; visitors see the host's aviary "exactly as it is."

- Friend-visited notifications off by default: The plan says the host gets "no push, no email, no in-product notification" by default, aligning visits with quiet observation rather than announcement.

### Performance, observability, and rollout

- Initial JS bundle under 2MB: The plan connects this to first paint and to the recorded-audio rejection, where recorded audio would collapse the needed bundle budget.

- Time to first bird visible under 500ms: This supports the first-frame requirement that birds appear already in motion and the quiet loading-state risk.

- 60fps idle motion: The plan calls this a runtime budget, ensuring continuous idle motion stays smooth on a 5-year-old laptop.

- No memory growth over 30 minutes: The plan treats this as a real CI test and ties it to procedural audio buffer reuse.

- Synthetic performance checks: The plan uses automated browsers from common geographies to check the aviary on a schedule and catch performance degradation.

- Aggregate-only Real User Monitoring: The rationale is observability without crossing the privacy boundary: timings and errors are collected, but "No per-bird state in telemetry."

- Simulation-tick p99 alarm above five seconds: The reason is early detection of degradation because the tick should take much less than that.

- Last-two-major-versions browser support: The plan rejects very old compatibility paths because the "cost-benefit doesn't justify bundle bloat."

- Matter-of-fact unsupported-browser surface: This follows the system-surface voice rule: explain what is needed without naturalist phrasing.

- Single deploy with no feature flags for v1: The plan says all features are load-bearing and releasing piecemeal would ship "an incomplete product."

- No gradual ramp on birds-per-aviary for v1: The reason is that 2 starter birds and max 7 are already the day-one shape, with 7 as the audio recognizability cap.
