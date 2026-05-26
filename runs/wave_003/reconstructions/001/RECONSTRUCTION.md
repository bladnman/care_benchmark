## System-level intent

- **v1 boundary and non-goals respect.** The plan frames the product as a bounded v1: a "single horizontal browser-based aviary" with explicit exclusions for "native app," "gamification/streaks/achievements/leaderboards," "Tamagotchi hunger/distress/decay mechanics," "social-network surfaces," push notifications, customizable scenes, multi-aviary accounts, recorded-audio fallbacks, and numerical personality exposure.
- **Recognizability over scale.** The plan says "Bird count caps at 7 because call signatures must remain individually recognizable," and rollout should "ramp birds-per-aviary gradually post-launch while monitoring recognizability." The same intent shows up in age-gated new birds.
- **Canonical server state over client mutation.** The plan repeats that the server tick is the "single writer" and "sole writer of personality/mood," clients "write only events," and "Client never mutates canonical state." This is tied to avoiding "last-write-wins" and preventing client mutation from silently corrupting drift.
- **Expressive drift without neglect punishment.** Personality drift is "presence-driven" and "monotonic toward expressive," with "no negative drift on neglect" and "monotonic upward only." The plan also specifies strict presence and a slow low-pass filter so drift is neither "Tamagotchi feel" nor "screensaver."
- **Strict presence as a product and calibration boundary.** Presence is a "triple-condition record" and must be "visible + focused + recent pointer/key activity." The risk section says lax presence would cause "presence-signal inflation" and "leaks drift speed across population."
- **Voice split: naturalist for the aviary, matter-of-fact for systems.** The plan assigns "naturalist" voice to aviary/notebook/narration/captions and "matter-of-fact" voice to auth/errors/settings. It says "Voice leakage" on error surfaces "erodes trust."
- **Procedural and live rather than canned.** The plan prefers procedural WebAudio, real-time variation, procedural assets, and live snapshots. It says recorded loops fail chorus, no recorded fallback audio files are allowed, and "silence fallback preferred over canned."
- **Accessibility as a designed v1 surface.** The plan ships screen-reader narration, reduced-motion, call captions, keyboard navigation, focus rings, and WCAG AA chrome in v1. The risk section says late narration or reduced-motion "would exclude users" and accessibility must be "designed surfaces, not parity checklists."
- **Privacy and aggregate-only observability.** The plan uses synthetic UUID partitioning, encrypted email, "no email-derived keys," aggregate-only RUM, no per-account/per-bird data, and "No per-bird telemetry ever enters analytics warehouse."

## Per-feature whys

**Scope - v1 Definition and Non-Goals Respect**

- Single horizontal browser-based aviary: NOT RECOVERABLE FROM PLAN
- 2-7 birds per account: The plan says the cap exists because "call signatures must remain individually recognizable"; later birds appear only on an "aviary-age schedule."
- Server-side simulation tick (~1/min): The plan ties this to the backend simulation service as the "single writer of personality state," running "independently of clients" and advancing state during offline periods.
- Presence-driven monotonic personality drift: The plan says presence-time has the "dominant weight," drift is "monotonic upward only," and there is "no negative drift on neglect."
- Mood transitions: The plan connects mood changes to "time-of-day, recent offers/listen-ins, bird-to-bird call response propagation, personality modulation," so mood carries state from time and interaction signals.
- Procedural WebAudio calls: The plan says procedural audio enables "real-time variation on every call," true chorus overlap, and avoids recorded loops that "fail chorus."
- Return-greeting: NOT RECOVERABLE FROM PLAN
- Listen-in: The plan says listen-in should rebalance the mix with the "focused bird up, others ambient; never full mute" and ramp the mix "never hard cut."
- Offers (seed/song/pool): The plan makes offers append-only interaction events that mood transitions read and drift uses as a secondary signal; it does not separately explain why seed, song, and pool are the offer set.
- Settle gesture: NOT RECOVERABLE FROM PLAN
- Field notebook: The plan says the notebook "generates and stores sparse naturalist entries from tick + event signals" and shares naturalist voice with narration.
- Magic-link auth: NOT RECOVERABLE FROM PLAN
- Multi-device sync via canonical server state: The plan says canonical server state gives "identical snapshots," avoids "last-write-wins," and means "no reconciliation logic needed."
- Visit invitations: The plan says visits are "read-only, opt-in, revocable, default-off," and the visit service is a "read-only snapshot proxy" with "no event recording from visitors."
- Screen-reader narration: The plan says narration is "running naturalist prose" from the same state snapshot, with priority on events, and late narration would "exclude users."
- Reduced-motion cross-fade mode: The plan says reduced-motion uses "cross-fade still-pose sequences + slowed color/day shifts" while preserving "identical mood/drift/notebook behavior."
- Call captions: The plan says captions are runtime prose from the call grammar, positioned near the bird, and become default-on when WebAudio is unavailable.
- WCAG AA chrome: The plan says chrome text must meet "WCAG AA contrast" and keyboard focus must have a visible high-contrast ring.
- Day/night + ambient weather: The plan routes local time and weather into mood transitions, palette shifts, and weather "mood dampening."
- Top-bar chrome that fades: NOT RECOVERABLE FROM PLAN

**Architecture - Service Shape and Client/Server Split**

- Frontend render pipeline: The plan keeps the frontend as browser-only scene rendering, WebAudio, event emitters, and snapshot interpolation so the client can present local motion while consuming server state.
- No local persistence of personality vectors: The plan says "No local persistence of personality vectors" and later says personality is server-only canonical, supporting the server-state boundary.
- Auth service with synthetic-UUID account IDs and revocable session tokens: The plan uses "synthetic-UUID account IDs" and "session tokens (revocable)" so accounts are not keyed from email and sessions can be revoked.
- Snapshot service: The plan says snapshots should be "small state snapshots" from canonical records and "CDN-edge friendly."
- Notebook service: The plan says this service generates and stores sparse naturalist entries from tick and event signals.
- Visit service: The plan says this service issues and revokes invites and proxies read-only snapshots with no visitor event recording.
- Account export (JSON): NOT RECOVERABLE FROM PLAN
- Soft-delete (30d): NOT RECOVERABLE FROM PLAN
- Email change verification: NOT RECOVERABLE FROM PLAN
- Render pipeline boundary: The plan says the client receives a deterministic snapshot and runs local idle micro-motion, procedural audio, and ornamentation while never mutating canonical state.
- Encrypted email and no email-derived keys: The plan says account state lives under synthetic UUID partitioning and "no email-derived keys," with encrypted email in the account record.
- Settings for reduced-motion, captions, and visit notifications: The plan includes these settings as opt-in surfaces aligned to accessibility and visit controls.
- Bird stable internal UUID: NOT RECOVERABLE FROM PLAN
- Fixed 6-species pool: NOT RECOVERABLE FROM PLAN
- Bird renameable user name: NOT RECOVERABLE FROM PLAN
- Personality vector dimensions: NOT RECOVERABLE FROM PLAN
- Interaction event append-only log: The plan says interaction events are append-only, order preserved, and consumed by the simulation tick to guarantee additive correctness.
- Visit invite token, expiry, and revoked flag: The plan includes token, `expires_at`, and revoked flag to support invite issuance, expiration, and revocation.

**Simulation Engine Design**

- Tick cadence ~60s: NOT RECOVERABLE FROM PLAN
- Drift low-pass filter: The plan says drift should show measurable instrument change after about one week, visible user change after about three weeks, and "No single-session visible drift."
- Mood persists across sessions: The plan says mood persists and the tick advances during offline periods, keeping state continuous across sessions.
- Bird-to-bird chorus emergence and mood contagion: The plan names "bird-to-bird call response propagation" in mood transitions and later summarizes this as "chorus emergence, mood contagion."
- Adoption with two starter birds and age-gated later birds: The plan starts with two birds and adds later birds on aviary-age gates, matching the recognizability cap and age schedule.

**Sync Model**

- Single canonical record per aviary: The plan says the server tick is the only writer, clients write only events, and this avoids last-write-wins and preserves additive correctness.
- Pull snapshot on tab visible, long gaps, and keepalive: NOT RECOVERABLE FROM PLAN
- Clients write only events: The plan says clients write only append-only events and never personality writes, keeping server deltas authoritative.
- Matter-of-fact conflict errors: The plan says magic-link replay and timeout conflicts surface as matter-of-fact errors, with "no naturalist phrasing."

**Frontend Rendering Pipeline**

- Three perch zones and responsive compression: The plan says responsive compression must preserve "all birds in frame."
- First frame live mid-action state with no wake-up animation: NOT RECOVERABLE FROM PLAN
- Loading state as quiet field with no spinner: NOT RECOVERABLE FROM PLAN
- Idle micro-motion mood-shaped: The plan says idle behaviors such as "preen/scan/tilt/fluff" are mood-shaped, with cross-fades only in reduced-motion.
- Subtle parallax plus ambient leaf/feather drift: NOT RECOVERABLE FROM PLAN
- Slow lighting and listen-in transitions: The plan says settle/evening lighting should be slow and listen-in mix ramp should "never hard cut."

**Audio Pipeline**

- Per-species motif library and call grammar scheduling: The plan says motifs vary by species and mood, while vocal_frequency shapes inter-call interval and timing/pitch.
- WebAudio unavailable fallback: The plan says WebAudio unavailable should produce "graceful silence + captions default-on," and prefers silence over canned fallback audio.
- No per-call allocation leaks: The plan ties this to performance: "60 fps idle" and no memory growth.

**Accessibility Surfaces**

- Keyboard navigation and focus ring: The plan requires "full tab/arrow/enter/escape navigation" and a "visible high-contrast focus ring."

**Performance Budgets and Observability**

- Bundle under 2 MB gzipped initial: The plan says this budget "drives procedural audio + procedural assets."
- Time-to-first-bird under 500 ms on mid-tier 4G mobile: NOT RECOVERABLE FROM PLAN
- Runtime 60 fps idle with zero memory growth: The plan specifies buffer reuse and "no retained DOM on scroll" to prevent memory growth over 30 minutes.
- Synthetic browser fleet and p99 tick latency alarm: The plan uses this observability to alarm when p99 tick latency reaches 5 seconds.
- Aggregate-only RUM with no per-account/per-bird data: The plan says RUM must be aggregate-only and excludes per-account and per-bird data.
- Browser support for last two major versions: NOT RECOVERABLE FROM PLAN

**Rollout**

- Gradual ramp of birds per aviary: The plan says to ramp gradually while "monitoring recognizability."
- Instrumentation from day one: The plan names request counts, snapshot latency, tick p99, render-frame timing, and audio errors so the rollout can monitor health with aggregate-only metrics.
- No per-bird telemetry in analytics warehouse: The plan states this as a privacy boundary enforced at metric definition.
