## System-level intent

1. **"Feels alive, not robotic" is the governing product test for architecture, rendering, audio, sync, and rollout.** The plan says every architectural choice is justified against "feels alive, not robotic," then carries that through first-frame birds "already in motion," idle micro-motion, procedural calls that "vary every time," "true chorus, not pre-mixed audio," mood nudges rather than snaps, and iterative listening-test passes for "Audio uncanniness."

2. **"Notice, never announce" means the product should observe and imply rather than surface mechanics.** This shows up in the naturalist-voice field notebook, screen-reader narration that reads as "observation rather than as a transcript," no notebook unread marker, offer cooldowns absorbed as "a quiet no-op," soft sky loading "not a spinner," and a visit notification toggle that is "off-by-default."

3. **The plan treats no-Tamagotchi / no-gamification as schema and infrastructure constraints, not just UI restraint.** It refuses "hunger/decay timers," "distress states," "death/reset," achievement tables, streak counters, visit-frequency tables, engagement scores, cross-account ranking statistics, notebook unread state, and milestone infrastructure. It explicitly says these are refused "at the schema level" so they cannot be "just exposed" later.

4. **Server-owned truth, client-owned rendering is a hard boundary.** The plan states the line exactly: "anything that determines what a bird's personality or mood is lives server-side; anything that determines how that state is rendered lives client-side." This is reinforced by "clients never tick; they pull snapshots and interpolate," server-authored personality/mood, client-side Canvas2D rendering, WebAudio synthesis, and client-only ornaments.

5. **Personality drift must be additive, monotonic, and structurally safe from last-write-wins.** The plan names the database-enforced write path: only the tick job writes `Bird.personality`; deltas are clamped to `>= 0`; per-account ticks are serialized; the ingest API role has no `UPDATE` grant on the personality column. The intent is to prevent "morning's drift silently deleted" and keep drift asymmetry intact.

6. **Birds should read as independent animals, not configured avatars or synchronized system events.** The plan uses "meeting an animal, not configuring an avatar" for starter birds, stable bird identity "never reassigned, never reused," per-bird jitter for daily reset, per-instance timing jitter so birds do not visibly sync, and client-randomized call timing so simultaneous sessions do not hear bit-identical calls.

7. **Accessibility is part of v1's designed experience, not a later compatibility layer.** The plan says narration, reduced-motion, and captions ship "with v1, not as a follow-up." Reduced-motion is "a parallel rendering path" and "different rendering of the same aviary," captions come from the same call grammar as audio, and screen-reader narration shares vocabulary/style rules with the notebook.

8. **Privacy and social limits are architectural properties.** The plan isolates telemetry so analytics has no credential to the simulation DB, makes visit sessions read-only by token claim, keeps raw email only in auth, and treats a visitor as "a scoped, read-only client." It calls the privacy boundary "an architectural property, not a policy."

9. **Matter-of-fact system surfaces and naturalist ambient surfaces must not drift into each other.** The plan says naturalist-vs-matter-of-fact voice split is enforced at the API error-message layer, while notebook entries, narration, captions, and ambient observation use naturalist prose. Error payloads are server-generated so clients "can't accidentally drift into naturalist phrasing for a system surface."

10. **Performance choices are justified as ways to preserve the quiet, alive first impression.** The plan connects <500ms time-to-first-bird to CDN-edge inlined snapshots, Canvas2D over WebGL, procedural audio instead of recorded files, no render-blocking non-critical assets, and memory controls. The rollout section says performance regressions are "effectively unfixable-after-the-fact" in their effect on early trust.

## Per-feature whys

### Scope

- **Single-user accounts, magic-link auth, one canonical aviary per account**: The one canonical aviary supports canonical server state and avoids client-side merge. Magic-link auth rationale is partly recoverable through account enumeration protection and matter-of-fact auth/session error handling.

- **Two starter birds at adoption**: The plan says this is "meeting an animal, not configuring an avatar"; starter species are server-selected, not user-chosen.

- **Server-driven offers of new species as the aviary ages, cap of seven**: Age-threshold offers avoid a separate "milestone" subsystem that would look like gamification-milestone machinery. Tuning data must remain operational, "not a feature." The specific cap of seven: NOT RECOVERABLE FROM PLAN.

- **Server-side simulation tick driving personality drift and mood**: The tick makes server-owned personality/mood real in code, prevents clients from ticking, centralizes additive deltas, and keeps mood persistent across ticks and sessions.

- **Procedural, client-synthesized calls, bird-to-bird call interaction, chorus mixing**: Procedural WebAudio avoids recorded files, keeps bundle size down, makes calls "vary every time," enables "true chorus, not pre-mixed audio," and avoids phase-cancellation artifacts from stacked recorded loops.

- **Single horizontal scene, three perch zones, day/night cycle anchored to local time, ambient weather, idle micro-motion, reduced-motion mode**: The single scene and Canvas2D choice are justified by a static-camera, no-pan visual complexity and 60fps/bundle budget. Three perch zones express mood/personality through front/middle/back placement. Day/night is client-side because it is "purely a render concern." Ambient ornaments are not simulation-tracked so the simulation stays canonical only for meaningful state. Idle micro-motion supports "already in motion." Reduced-motion is a designed render mode so reduced-motion users are not told the product "wasn't for them."

- **Return-greeting**: Its specific rationale is mostly accessibility-related: it is one of the user-initiated priority events allowed to briefly use an `assertive` ARIA nudge. Additional product rationale: NOT RECOVERABLE FROM PLAN.

- **Listen-in**: Listen-in contributes to `social_warmth` and `vocal_frequency`; the audio mix ramps the focused bird up and others down with no hard cut and no full silence, preserving the "slow rise/slow drop" feel.

- **Offer (seed / song fragment / still pool)**: Offers are a small interaction signal: accepted offers bias mood toward `content`, accepted offers near a bird add a small `curiosity` delta, and any nearby offer gives a smaller `boldness` delta. During cooldown, failure is hidden because "you can't offer yet" would be a system-voice intrusion.

- **Settle with 5s undo**: The plan says settle has no directional drift and closes the presence window for tick accounting. The reason for the 5s undo specifically: NOT RECOVERABLE FROM PLAN.

- **Field notebook: sparse, auto-generated, naturalist-voice, read-only**: The notebook is not a generic event log; entries are prose, not typed events. Read-only/no edit/delete/annotate preserves that they are observations. Sparse generation and no unread count avoid the system tracking cadence and surfacing it back.

- **Presence accounting with visibility + focus + recent pointer/key activity, conjunctively**: This implements "idle attention is itself an interaction" while requiring the tab to be visible, focused, and active. Ping intervals bound event-log volume, and the server derives presence-time without inflating missed pings.

- **Multi-device sync via canonical server state, no client-side merge**: Because personality/mood are server-owned and additive, there is "no merge step"; laptop and phone pull the same `Bird` rows. This prevents silent drift loss from last-write-wins.

- **Visit-invitation: per-invite opt-in, read-only ambient visits, revocable, visit log, off-by-default friend-visited notification toggle**: The visit feature is intentionally a read-only consumer of the same snapshot API, with no separate social infrastructure. Opt-in, revocation, and default-off notification keep it from becoming a social-network surface or growth mechanic.

- **Screen-reader narration**: It gives naturalist prose, not a state-list, and reads as "observation rather than as a transcript." Sharing phrase-construction with notebook vocabulary/style rules guarantees voice continuity.

- **Reduced-motion as its own designed render mode**: A separate render strategy makes reduced-motion a different rendering of the same aviary, not a CSS afterthought, and prevents the post-launch message that reduced-motion users were not included.

- **Call captions generated from the call grammar at runtime**: Captions derive from the same motif selection that drives synthesis, guaranteeing caption-matches-what-played.

- **WCAG AA contrast on chrome**: Contrast on top-bar, settings, and error text is enforced with design tokens and build-time lint because those surfaces carry user copy.

- **Full keyboard navigation**: It ensures the top bar, birds, listen-in, offers, and settle are reachable without pointer input. The dynamic focus ring is designed because the aviary background changes.

- **Performance budgets**: Initial bundle, time-to-first-bird, frame rate, and memory growth preserve trust and the "feels alive" claim. The plan ties each budget to an implementation lever.

- **Account export and soft-then-hard account deletion**: The 30-day soft-delete/restore path gives "I changed my mind" recovery before hard deletion. The emailed export link/job rationale beyond account lifecycle: NOT RECOVERABLE FROM PLAN.

### Out of v1

- **No native apps or app-shell abstraction layer**: The web client is built as a web client because a cross-platform-ready core would cost render-pipeline quality the plan says it "can't spend."

- **No gamification surfaces of any kind**: Even internal admin-only aggregation is refused because internal surfaces can become user-facing "under product pressure six months from now."

- **No Tamagotchi mechanics**: No hunger, decay, distress, death, or reset because neglect must not punish the birds and the product refuses Tamagotchi dynamics.

- **No social-network surfaces beyond visit affordance**: Profiles, follows, feeds, discovery, comments, leaderboards, and aggregate metrics are excluded so social does not become ranking or network infrastructure.

- **No shared/multi-aviary accounts**: NOT RECOVERABLE FROM PLAN.

- **No payments**: NOT RECOVERABLE FROM PLAN.

### Architecture

- **Auth service**: It owns raw email because raw email is isolated to the auth service's account table; magic-link issuance/consumption, session lifecycle, account lifecycle, and email-change verification belong there.

- **Simulation service**: It is the load-bearing service and built/tested first because it owns canonical aviary records, tick correctness, personality vectors, mood, perch/position, weather/time-of-day derived state, and event ingest.

- **Social service**: It owns visits so visitor tokens can be read-only and scoped to a host account. Reading through the same API as the host makes there structurally no write path for visitors.

- **Notebook service**: It is decoupled so notebook prose generation can iterate independently of tick correctness-critical code and so it consumes summaries rather than raw event-log schema.

- **Edge/BFF layer**: It delivers small state snapshots from the CDN edge alongside the HTML shell to reach <500ms time-to-first-bird, aggregates top-bar data without unread state, and proxies interaction writes.

- **Telemetry/observability isolated pipeline**: It never reads the simulation database directly so aggregate telemetry cannot carry bird state or per-account interaction history.

- **Authoritative layer and interpolation/render layer**: The split lets the first frame render "already in motion" while keeping client ornaments from becoming server truth.

### Data model

- **Server-generated synthetic UUIDs**: They prevent email-derived identifiers outside auth, matching the "single most important boring detail."

- **Account settings**: Reduced motion, captions, visit notifications, and audio settings persist user preferences. Visit notifications default false per social optionality. Other setting rationale: partially recoverable from the surfaces they control.

- **Session device label and revocation**: The device label supports a user-visible device-revocation UI. Session revocation supports account control.

- **Aviary created_at**: It anchors "aviary age" for third-bird-and-beyond pacing.

- **Weather state**: Server-owned weather presence keeps weather consistent across devices/visitors while client visuals remain ornamental.

- **Stable Bird identity**: A bird id is "never reassigned, never reused" to preserve bird identity.

- **Bird name renameable with no effect on personality/mood/call**: Renaming lets the user name birds without turning names into simulation inputs.

- **Personality JSON on Bird row**: Personality is read/written together at tick time and never queried independently across birds, so JSON columns are simpler than normalized tables.

- **Mood on Bird row**: Mood persists across ticks and sessions and guarantees it does not reset on tab open.

- **Perch zone**: It renders personality/mood spatially: boldness pulls front, wary mood pulls back.

- **Current animation hint**: It is a shared contract between full-motion and reduced-motion renderers, and build checks can ensure every emitted hint has both renderings.

- **Last offer accepted timestamp**: It supports per-bird cooldown tracking. Rationale for per-bird rather than global cooldown: NOT RECOVERABLE FROM PLAN.

- **InteractionEvent append-only log**: It is the only path by which client behavior reaches personality state, making client writes auditable and preventing direct personality/mood writes.

- **NotebookEntry with prose and no exposed kind/category**: Entries are prose, not typed events, matching "not generic event logs."

- **VisitInvite and VisitSession**: They support one-time invitation, opt-in active visits, revocation, expiry, and refresh without re-consuming the same link.

- **VisitLog as read projection**: It avoids a separate write path.

- **Absent streak/visit-frequency/engagement/ranking fields**: Their absence prevents later UI-only exposure of gamification surfaces.

- **Absent happiness/decay/hunger field**: This blocks Tamagotchi mechanics at schema level.

- **No personality-history table exposed to clients**: Calibration data can exist internally without exposing numerical personality vectors or exporting them.

### API surface

- **Server-generated error messages in matter-of-fact voice**: Clients render typed error responses verbatim so system surfaces do not drift into naturalist phrasing.

- **`POST /auth/magic-link` always returns 202**: This avoids account enumeration.

- **`POST /auth/magic-link/consume`**: Expired/used links return matter-of-fact errors, preserving the system-surface voice split.

- **`POST /auth/sessions/:id/revoke` and `GET /auth/sessions`**: They support the device-revocation UI.

- **`POST /auth/email-change`**: NOT RECOVERABLE FROM PLAN.

- **`POST /account/delete` and `POST /account/restore`**: They implement soft deletion and "I changed my mind" recovery before `hard_delete_at`.

- **`POST /account/export`**: NOT RECOVERABLE FROM PLAN.

- **`GET /aviary/snapshot`**: It is the core read endpoint and the shared host/visitor read path. It carries server time, tick time, weather, birds, and derived render hints so clients can render without raw personality vectors.

- **Omitting notebook unread state from snapshot**: An unread badge is treated as the same forbidden pattern of tracking cadence and surfacing it back.

- **`personality_render_hints` instead of raw vectors**: This makes "personality vector is never exposed numerically" a wire-format guarantee, not a UI omission.

- **Snapshot polling via visibility-change, long-frame-gap, and low-frequency keepalive**: These pulls keep state fresh across visibility, suspend/resume, and steady visible use without clients becoming authoritative.

- **`POST /events`**: One append-only endpoint is the only client write path that can influence personality/mood. Visitor tokens get 403 to preserve read-only visits.

- **Presence ping event**: It lets the server derive presence-time from observed inter-ping gaps while bounding client event volume.

- **Listen-in start/end events**: They let duration be computed server-side and fall back to received-at gaps if the end is missing.

- **Offer events with quiet cooldown no-op**: Cooldown is enforced server-side without surfacing a "you can't offer yet" system intrusion.

- **Settle and settle_undo events**: They are account-level with no bird id. The rationale for undo as an event beyond the interaction existing: NOT RECOVERABLE FROM PLAN.

- **`GET /offers/catalog`**: Static offer metadata ships in the initial bundle if small to avoid a render-blocking fetch.

- **Notebook cursor pagination**: Newest-first infinite scroll supports scroll-back without unread/new markers.

- **Visit invite/revoke/log/consume/snapshot endpoints**: They create, revoke, log, and consume read-only visits. Implementing visitor snapshot as the same endpoint guarantees visitors and hosts cannot see different data by convention drift.

### Presence accounting

- **Three-signal conjunction**: Visibility, focus, and recent pointer/key activity ensure presence means active attention to the tab.

- **Pointer throttling and 30s pings**: They avoid event-volume issues and bound write volume.

- **Server-distributed activity window**: The exact window can be recalibrated without a client release.

- **Presence accrues anywhere in-app**: The plan says presence is about "the tab," not "the scene"; route-gating would make users keep a specific route open to "count," which reads like a disguised engagement mechanic.

### Simulation engine

- **Queue-driven per-account tick pool**: It prevents tick latency from degrading as accounts grow and supports p99 5s alarms per account.

- **Jittered tick cadence**: It avoids thundering-herd load.

- **Event ordering by received_at**: Client clocks are not trusted for ordering.

- **Presence-time computation from ping gaps**: It measures observed presence without inflating network blips.

- **Personality deltas from presence, listen-in, offer, settle**: The weighting follows the stated order: presence-time dominant, listen-in affecting social/vocal traits, offers affecting curiosity/boldness, settle closing accounting without directional drift.

- **Monotonicity clamp**: It prevents any negative-drift path, including future signals accidentally introducing one.

- **DB-role restriction on personality updates**: It turns "no last-write-wins for personality state" into a database-enforced guarantee.

- **Per-account tick serialization**: It prevents tick races from silently deleting drift.

- **Mood state machine**: Mood responds to recent interactions, time-of-day, weather, and personality while neglect gently drifts ambient/drowsy rather than wary.

- **Timezone offset stored from snapshot requests**: The simulation only needs local hour, and storing offset avoids a timezone-database dependency.

- **Mood daily reset as jittered nudge**: Jitter prevents birds resetting in unison as a "system event"; nudging avoids an instantaneous snap users could catch.

- **Calibration harness**: It lets engineers tune delta scaling to 1-week instrument and 3-week visible targets without exposing debug views or numerical personality.

- **Call-grammar motif library**: Authored data rather than audio files keeps calls procedural, varied, and bundle-friendly.

- **Personality-shaped call synthesis**: It makes two birds of the same species distinguishable before mood.

- **Per-call jitter**: It is the concrete mechanism behind "calls vary every time."

- **Listen-in gain ramps**: They create slow rise/slow drop, no hard cut, and no full silence.

- **Chorus from overlapping independent calls**: It avoids pre-mixed loops and recorded-loop phase artifacts.

- **Bird-to-bird response probability server-side, timing client-side**: It keeps response probability consistent with personality/mood while avoiding bit-identical timing across sessions.

- **Idle motion state machine**: Server hints keep motion tied to mood/personality state, while client jitter prevents visible synchronization.

### Sync model

- **HTML shell plus inlined initial snapshot**: It avoids a second round trip for first-frame data and supports the <500ms budget.

- **Render-layer phase offset on cold load**: It makes first paint show birds mid-cycle rather than neutral.

- **Quiet-field loading state**: It avoids a spinner and keeps loading consistent with the product voice.

- **First-bird fly-in only after genuine adoption**: Entrance animation is reserved for the case where a bird has actually just arrived.

- **Steady-state interpolation**: The client visually tweens between server points but never computes new authoritative state.

- **Device-session-keyed listen-in pairing**: It prevents cross-device start/end pairing bugs; rare double-counting is the safer failure because drift is monotonic and bounded.

### Frontend rendering pipeline

- **Canvas2D sprite approach**: Full WebGL is unnecessary for static-camera, no-pan scene and would cost bundle budget; Canvas2D is sufficient for 60fps.

- **Parallax depth by scale/desaturation/blur**: It gives front/middle/back depth without true 3D. More detailed rationale: NOT RECOVERABLE FROM PLAN.

- **Perch-to-perch tweening**: It avoids teleporting.

- **Mood pose cross-fades**: They prevent birds from snapping between moods.

- **Reduced-motion cross-fade renderer**: It consumes the same hints/mood but outputs authored still poses with ornaments disabled and slowed color transitions.

- **OS reduced-motion handling**: Respecting `prefers-reduced-motion` as authoritative is called standard web accessibility practice, unless explicit account override is set.

- **Day/night palette client-side**: It is a pure rendering concern and independent of server tick cadence.

- **Ambient ornaments client-side**: Leaf/feather positions and timings have no server state because they are ornament, not simulation.

- **Empty-aviary state reuses quiet-field loading component**: Reuse keeps the empty state quiet and consistent; first-bird fly-in follows adoption.

### Audio pipeline

- **One AudioContext, per-bird GainNode, shared bus, master gain**: It supports bird-specific mixing and account-level audio enablement.

- **Linear gain automation**: It guarantees slow rise/slow drop as an audio-graph property.

- **Buffer/node reuse and disconnection**: It addresses the no-memory-growth budget because connected WebAudio nodes are not garbage collected merely by going out of scope.

- **No recorded-audio fallback**: Graceful silence plus captions avoids the temptation to enable recorded audio later and keeps the procedural/no-recorded-file premise intact.

- **Session-only captions fallback when WebAudio unavailable**: It makes the app usable in graceful silence without overwriting persisted account settings.

### Accessibility surfaces

- **ARIA live region narration cadence**: `polite` updates every 30-60s read as ambient observation; `assertive` is reserved for user-initiated priority events.

- **Narration from snapshot + render-layer state**: It avoids literal event-log transcripts and aligns narration with what the user can observe.

- **Keyboard order through top bar and birds**: It gives complete keyboard navigation in the same interaction space as the visual app.

- **Two-layer focus outline based on brightness**: It remains visible against dynamic bright and dim aviary backgrounds.

- **Captions anchored near calling bird**: They keep naturalist phrasing connected to the actual bird/call source. Additional rationale: caption-matches-what-played from the audio section.

- **Build-time contrast lint**: It catches token contrast problems in CI because chrome/settings/error text use controlled design tokens.

### Performance budgets and observability

- **Initial JS bundle under 2MB**: Procedural audio, code-splitting, compact assets, and no WebGL engine dependency keep the bundle small.

- **Time to first bird under 500ms**: CDN-edge inlined snapshots and non-blocking assets make the first bird visible quickly.

- **60fps idle motion**: Canvas2D and bounded per-frame work keep motion smooth on a 5-year-old laptop.

- **No memory growth over 30 minutes**: WebAudio lifecycle, notebook virtualization, and bounded worker/audio contexts prevent growth.

- **Synthetic checks**: They measure first-bird timing and frame timing from common geographies and alert on regressions.

- **Aggregate RUM**: It tracks timing/errors/tick latency while excluding per-bird state and per-account history.

- **Tick latency alarm**: p99 above 5s catches simulation service degradation.

- **Typed aggregate-emission telemetry**: The emission function only accepts scalar metric fields, so analytics cannot carry personality vectors or event payloads.

### Rollout

- **Simulation service first**: Tick correctness is the critical path and must be validated before client work depends on it.

- **Accessibility ships with v1**: The plan says post-launch reduced-motion would tell reduced-motion users the product was not for them.

- **Instrumentation from day one**: Drift calibration and performance regressions damage early trust and are effectively unfixable-after-the-fact in user perception.

- **Visit feature in v1 with no separate ramp**: It is a read-only consumer of the same snapshot API and has no separate social infrastructure phase.

### Risks and mitigations

- **Drift calibration risk**: Too fast becomes "Tamagotchi-with-extra-steps"; too slow becomes a screensaver. The calibration harness gates constants and only scale can be tuned, never sign or negative drift.

- **Sync correctness risk**: Failures are silent, so the plan uses per-account tick serialization, DB-level write restriction, and device-session-keyed event pairing as structural guarantees.

- **Audio uncanniness risk**: Procedural calls need listening-test passes because engineering correctness alone will not catch the perceptual/design risk.

- **Accessibility regression risk**: Shared `current_animation_hint` and build-time registry checks ensure every new normal-motion hint has reduced-motion and narration support.

- **Gamification-foothold risk**: The plan refuses engagement tables, unread state, and ranking computation so later gamification requires new schema and computation, not a config flip.
