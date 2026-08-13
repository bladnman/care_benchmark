## System-level intent

- Ambient, non-gamified, non-punitive care. This shows up in the v1 exclusions of "scores, streaks, badges, levels," "death, hunger, distress," and "punishment for absence"; in the invariant that "Drift is monotonic toward expressive"; in the drift rule "No negative terms. No neglect term."; and in expressiveness as "neglect is quiet, not punished."

- The aviary should feel already alive, not like an app loading. This shows up in the CI-level rule "First painted frame is mid-action or a quiet field -- never a spinner"; in the "first-frame 'already running' scene"; in load behavior that paints before fonts/audio/notebook; and in the risk named "Spinner culture."

- Canonical state belongs to the server and the worker, while clients only render and append events. This shows up in "server is the only writer of canonical state," "Clients never tick," "Clients never write personality or mood," "There is no merge. There is no LWW," and the forbidden code paths for client vector writes.

- Presence is evidence, not mere page-open engagement. This shows up in the invariant requiring `visibilityState === 'visible'`, window focus, and recent input; in the "three-signal presence" mitigation; in ping interpolation where "A single ping = 0s"; and in the rejection of a "tab open" shortcut.

- The product voice is split: naturalist on aviary surfaces, matter-of-fact on system surfaces. This shows up in the invariant "Naturalist voice on product surfaces; matter-of-fact voice on auth, errors, sync, account, and a11y-settings"; in "No naturalist errors"; in notebook "lowercase naturalist" prose; and in the ban on error copy "in warbler-speak."

- The user should experience birds through behavior, sound, color, and prose, not through numbers. This shows up in "No personality numbers anywhere," "Visual/audio/narration are the only live expressions," "Do not expose mood names in UI," and banned strings such as mood labels, trait numbers, and stats.

- Privacy is a product boundary, not only an implementation detail. This shows up in synthetic UUIDs, encrypted email, blind indexes, "Email lives in one encrypted column and nowhere else," "Per-bird interaction data never enters aggregate telemetry," separate simulation and metrics credentials, and the rule against metrics that "reconstruct a relationship."

- Social behavior is constrained to opt-in, read-only visits, not a network. This shows up in "per-invite opt-in," "read-only ambient view," "notifications off by default," no profiles/follows/discovery/comments/chat/co-presence, "Visitor snapshots never include a greeting plan," and "visit endpoint cannot POST events."

- Accessibility is part of v1, not a follow-up. This shows up in "Accessibility shipping on day one," "Ship with v1. Not a follow-up," reduced motion as a "first-class aesthetic," captions generated from the call grammar, and "a11y launch criterion equal to visual launch."

- Performance budgets are product guardrails. This shows up in first-bird under 500ms, initial JS under 2MB, 60fps idle motion, no heap growth over 30 minutes, low single-digit KB snapshots, and the rollout sequence reserving "Perf gates" before dogfood.

- Audio should be procedural, recognizable, and alive rather than looped. This shows up in "Never loop a buffer as 'the call,'" species motif libraries, sticky per-bird timbre from `hash(bird.id)`, independent chorus voices, and the recognizability test where listeners identify birds after exposure.

- Decisions should be validated through staging, CI, and qualitative playtest, not product-surface experiments that violate non-goals. This shows up in compressed-time drift tests, banned feature flags such as "faster drift" and "welcome toast," staging-only clock offsets, and retuning only "against calibration tests."

## Per-feature whys

### Scope and product surface

- **Browser-only SPA**: NOT RECOVERABLE FROM PLAN.

- **Single-user account, one canonical aviary per account**: NOT RECOVERABLE FROM PLAN.

- **Email magic-link auth**: NOT RECOVERABLE FROM PLAN.

- **Per-device revocable sessions**: NOT RECOVERABLE FROM PLAN.

- **Two starter birds at signup**: The plan says the starter pair is picked with "dissimilar motif families and silhouettes" so "the first chorus is distinguishable." The post-signup empty-aviary moment exists only once, then each starter can fly in once.

- **Hard cap of seven birds**: The plan uses the cap as an audio and social boundary: the audio risk names "seven birds becoming mush," and the cap also supports the non-goal of no collecting or public flock status.

- **Additional birds gated by aviary age, not engagement**: The rationale is to avoid gamification and engagement pressure. The plan says adoption is from `aviary.created_at`, uses quiet naturalist copy, is "not a modal, not a toast," avoids "unlock," and says there is "No marketing of 'collect all six.'"

- **Server-owned personality vectors, mood, perch intent, weather, and notebook generation**: The rationale is canonical sync and protection from client corruption. The plan repeats that clients append events but never write personality or mood, and the worker is the only updater for personality, mood, perch intent, expressiveness, weather, notebook, and adoption clocks.

- **Client-owned interpolation, idle micro-motion, procedural WebAudio calls, and first-frame scene**: The rationale is that these are ephemeral rendering responsibilities. The plan says client state is "never authoritative," the renderer writes back into no canonical fields, and visuals must not wait for WebAudio.

- **Presence / idle attention**: The rationale is "honest presence." The plan requires visible state, focus, and recent pointer/key activity, drops inconsistent pings, counts only valid coverage, and says this stops "one accidental mouse move" from minting drift.

- **Listen-in**: The rationale is focus without erasing the ambient place. The focused bird gain eases up, other birds ease down only to a floor, "never zero"; the plan says hard cuts are a bug and the floor keeps the place from becoming a DAW.

- **Offers: seed, song fragment, still pool**: The rationale is a quiet interaction that can create mood-shaped reactions and positive drift. The plan connects seed to approach/wait/ignore, song to join/quiet/call-against, pool to drink/bathe/watch, and maps accepted or nearby offers to curiosity, boldness, and vocal behavior.

- **Settle plus 5s undo**: The rationale is local atmosphere rather than trait manipulation. The plan says there is no settled column, settle lighting is session-local, settle has "zero personality delta," and undo reverses the lighting lerp within five seconds.

- **Field notebook**: The rationale is sparse naturalist observation, not a stat log or user journal. The plan makes it read-only, prose-only, lower-case naturalist, sparse, banned from words like "you," "session," "streak," and "unlocked," and says trigger is not exported to the product surface.

- **Rename**: The rationale is that naming must not affect personality. The API accepts `{ name }` only, rejects unknown keys, and says `rename.payload.name` only; "personality untouched."

- **Visit invitations**: The rationale is opt-in ambient sharing without social-network mechanics. The plan uses per-invite opt-in, read-only visitor snapshots, revocation, expiry, no visitor events, no visitor presence, no greeting, no notebook, no host actions, and no host notification unless `visit_notify_email` is true.

- **Accessibility shipping on day one**: The rationale is that accessibility is a launch criterion. The plan names naturalist screen-reader narration, reduced motion, call captions, WCAG AA chrome, and keyboard paths as v1 scope, then says "Ship with v1. Not a follow-up."

- **Account export**: The rationale is account control and privacy. The plan provides JSON export by emailed signed URL, keeps export blobs short-TTL, and limits user-facing notebook export to `written_at` and `prose`.

- **Soft-delete then hard-delete**: The rationale is reversible account deletion followed by real removal. The plan gives a 30-day soft-delete marker, undelete while before `hard_delete_after`, and a privacy mitigation that export/delete "actually drops events and vectors."

- **Gamification exclusions**: The rationale is to keep the aviary from becoming scores, streaks, milestones, or success dashboards. The plan excludes scores/streaks/badges/levels and later excludes "engagement dashboards, DAU-as-success, streak-like funnels."

- **Tamagotchi mechanic exclusions**: The rationale is non-punitive care. The plan excludes death, hunger, distress, decaying happiness, and punishment for absence; the drift and expressiveness sections make quietness separate from negative trait drift.

- **Social-network surface exclusions**: The rationale is to prevent social creep. The plan excludes profiles, follows, discovery, comments, chat, avatars, co-presence, leaderboards, public aviaries, and explicitly says no public list tables in the schema.

- **Push notifications excluded**: The rationale is to avoid aviary pings. The plan says no push notifications of any kind, and makes opt-in visit email the sole exception, off by default and not shown in onboarding.

- **Personality numbers hidden**: The rationale is that birds should express traits live, not expose stats. The plan bans personality numbers in production UI and debug UI, hides raw mood names, and says visual/audio/narration are the only live expressions.

- **Recorded-audio fallback excluded**: The rationale is procedural audio and captions instead of canned sound. The plan says WebAudio missing means silence plus captions, no recorded files, and "Never loop a buffer as 'the call.'"

### Architecture, API, and data

- **Three deployable units: api, sim-worker, mailer**: NOT RECOVERABLE FROM PLAN.

- **Postgres canonical state plus append-only event log**: The rationale is canonical storage and replayable worker consumption. Postgres owns canonical state and the event log; events are append-only, consumed by tick, and not deleted except hard-delete.

- **Redis for sessions, magic links, rate limits, locks, and snapshot cache**: The rationale is short-lived operational state. The plan uses Redis for token lookup, single-use magic links, rate limits, tick locks, and short-lived snapshot cache rather than canonical bird state.

- **Object store for export JSON blobs**: The rationale is short-lived account export delivery. The plan stores export blobs with signed short-TTL URLs and emails the link.

- **CDN / edge static assets and snapshot bootstrap**: The rationale is first-bird speed. Static assets and the HTML shell come from CDN/edge, and authenticated HTML may include a last-known snapshot so the client can paint before other resources.

- **No Kafka, no analytics warehouse reader, no client-to-client channel**: The rationale is lower sync and privacy surface. The plan keeps one data plane, no warehouse reader on simulation DB, and no client-to-client channel.

- **Client / server split**: The rationale is authoritative server simulation with client-only presentation. The server owns account, aviary age, identities, vectors, mood, weather, offers, notebook, visits, eligibility, and greeting plan; the client owns interpolation, audio, presence detector, captions, and local settle lighting.

- **Render pipeline boundary**: The rationale is swapability and no canonical writes. SceneState becomes SceneGraph, then visual renderer, overlay, audio, narration, and captions; VisualRenderer is swappable and "No renderer writes back into canonical fields."

- **Trust and privacy boundary**: The rationale is separating simulation data from telemetry and PII. The plan uses separate credentials/schemas, operational allowlists, encrypted email, and a distinct read-only visit capability.

- **Synthetic UUIDs plus encrypted email and blind index**: The rationale is avoiding email as an identifier. The plan says IDs are the only log/lock/partition identifiers, email is encrypted, lookup uses a blind index, and email appears nowhere in logs, keys, or telemetry dimensions.

- **Append-only `interaction_events`**: The rationale is idempotent event ingest for the worker. The plan uses unique `(session_id, client_event_id)`, schema-validated payloads, tick consumption, and forbids updating event rows except `consumed_at`.

- **Notebook entry storage**: The rationale is product prose without exposing internal triggers. Entries retain prose indefinitely until hard-delete; `trigger` is internal and omitted from user-facing export.

- **Visit invites and visit sessions**: The rationale is revocable read-only sharing with silent host logs. Invites hash tokens, expire after 30 days, can be revoked, and sessions store approximate duration without presence events.

- **Species catalog and starter-pair selection**: The rationale is coherent woodland variety and distinguishable first sound/shape. The catalog has six species with silhouettes, palettes, call grammars, perch priors, and nocturnal status; starters are dissimilar.

- **Personality seed**: The rationale is room for slow growth. The plan samples mid-low initial traits so "weeks of presence have room to move toward expressive" and never persists client-side seeds as truth.

- **No raw vectors or `expressiveness` in live client resources**: The rationale is to keep stats off the product surface. Export may include vectors, but live expression is through visuals, audio, and narration.

- **Host snapshot shape**: The rationale is to give renderable state without exposing raw traits. It sends weather, birds, perch, baked plumage colors, call hints, greeting plan, adoption status, narration, and settle suggestion; raw saturation is avoided by decision to send baked palette colors.

- **Event API strict validation**: The rationale is preventing client personality writes and abusive payloads. Unknown keys are rejected, cooldown is server-enforced, rate limits apply, and no personality fields are accepted.

- **Presence pings and dedicated `presence_end`**: The rationale is honest coverage with a clear falling edge. The activity window is long enough that "still watching counts," but pings older than the activity window are dropped.

- **Visit flow and visitor chrome**: The rationale is ambient viewing without announcing social state. The visitor cannot post events, cannot listen-in/offer/settle/read notebook, and has empty chrome except accessibility settings and leave.

- **Snapshot pull triggers and ETag / `tick_version`**: The rationale is freshness at visibility edges and efficient interpolation. Pulls happen on navigation, visible changes, bfcache resume, sleep gaps, keepalive, and read-your-writes; unchanged snapshots return 304.

- **Opt-in visit email exception**: The rationale is resolving the "no email about the aviary" ambiguity. Visit email is the sole exception, off by default, absent from onboarding, not push, and matter-of-fact.

### Simulation and sync

- **Tick loop and catch-up**: The rationale is server-side continuity without invented presence. The worker ticks about every 60 seconds, catches up real timeline weather/time-of-day, caps catch-up compute, and applies drift only from presence already in the log.

- **Presence integration**: The rationale is not counting isolated or overlapping noise as attention. Valid ping coverage is interpolated only between close pings, a single ping counts 0s, and presence_end is optional because coverage simply stops.

- **Drift function**: The rationale is slow monotonic expression. All weights are non-negative, there is no neglect term, and scripted honest presence over days must move traits enough to notice without exposing a demo-speed shortcut.

- **Daily trait delta cap**: The rationale is pacing. A cap of 0.008 per trait per UTC day "stops a 6-hour sit from skipping the week."

- **Expressiveness**: The rationale is "ambient quietness" without negative trait drift. It rises with recent honest presence, decays slowly toward a living floor, and scales greeting/call/viewer attention so neglect is quiet but not punished.

- **Mood transitions**: The rationale is persistent, contextual behavior rather than session reset. Mood is modulated by timezone, weather, offers, nearby wary birds, boldness, and curiosity; there is no snap-to-content on snapshot.

- **Perch intent**: The rationale is visible mood and boldness through position without restlessness. Wary/low boldness moves back, content/high boldness moves front, drowsy sits middle/back, curious biases toward offers, and birds may stay on the same perch for many minutes.

- **Bird-to-bird behavior and chorus**: The rationale is social texture inside the aviary. Recent calls can bias warm/vocal birds to respond, wary birds can pressure others, and chorus intent is rolled on the worker so clients reconstruct the same overlap.

- **Call-grammar runtime split**: The rationale is consistent, lightweight procedural sound. The server outputs seeds and hints, while the client synthesizes motif sequences; identity comes from species family and per-bird sticky timbre, not drifting trait labels.

- **Greeting plan**: The rationale is a host-only welcome that cannot replay or become a notification. It appears on host visibility edges, is suppressed for 90 seconds after emission, ranks birds by boldness/expressiveness/mood, staggers close birds, and is absent from visitor snapshots.

- **Offer cooldown**: The rationale is "Functional, not punitive." A four-minute shared per-bird cooldown prevents repeated offer reactions while extra offers are acknowledged as no-ops rather than treated as punishment.

- **Weather: short rains and wind passages**: NOT RECOVERABLE FROM PLAN.

- **Notebook authoring**: The rationale is sparse, observational memory. The generator runs only every 2.5 days or for noteworthy events, has a hard max of three entries per seven days, uses compositional naturalist templates, and has banned-token snapshot tests.

- **Adoption pacing**: The rationale is quiet long-term pacing without "unlock" language. Bird availability comes from aviary age, appears as a quiet top-bar affordance, uses naturalist copy, and the system picks species server-side.

- **Single-writer sync model**: The rationale is to avoid merge conflicts and vector corruption. Only the worker changes personality/mood/perch/expressiveness/weather/notebook/adoption clocks; api inserts events and updates non-sim account settings.

- **Multi-device union of presence seconds**: The rationale is allowing rare concurrent devices without double-counting attention. Overlapping presence is OR'd, not summed; listen-in can apply twice but daily caps bound it, avoiding co-presence machinery.

- **Preventing classic overwrite**: The rationale is protecting vectors from caches and LWW merges. The plan forbids client vector patches, localStorage source-of-truth vectors, reconcile averages, and direct client-set updates to bird traits.

- **Snapshot cache**: The rationale is efficient keepalives while preserving read-your-writes. Cache invalidates on tick/name/adopt, serves 304s, and is busted or patched after event posts that should be visible immediately.

- **Conflict / error surfaces**: The rationale is simple recovery without charming sync language. Since personality conflicts are impossible by construction, visible failures are auth/session/snapshot/visit errors, copy is matter-of-fact, and reload is recovery.

- **Offline behavior**: The rationale is that the server cannot prove offline presence and the client must not run a local sim. Presence drops while unreachable, mutations retry only briefly, and reconnect pulls a fresh snapshot instead of replaying local simulation.

### Frontend, audio, accessibility, performance, and rollout

- **TypeScript/Vite with Canvas 2D and DOM focus overlay**: The rationale is performance plus accessible hit targets. Canvas gives 60fps control and no layout thrash, while a transparent DOM overlay provides click/tap/focus.

- **Scene composition and responsive perches**: The rationale is calm naturalist depth without losing birds. The plan uses three depth planes, subtle parallax, fixed anchors that reflow, fully on-screen hit rects, and "Never crop a bird."

- **First frame / load**: The rationale is immediate aliveness. The client paints sky and snapshot birds before fonts/audio/notebook; missing snapshot shows quiet field, no spinner or logo pulse; pose seeds start birds mid-cycle.

- **Idle micro-motion**: The rationale is that birds are "Never fully still" while visible. Motion expresses mood through scan/preen/tilt/sit/head snaps, keyed by personality in timing rather than stat overlays.

- **Transitions**: The rationale is soft continuity. Perch moves ease over 4-8s, reduced motion crossfades, day/night uses continuous palette lerp, settle forces an evening lerp, and weather fades without flash.

- **Reduced-motion renderer**: The rationale is first-class aesthetics rather than removing all animation. It uses the same SceneGraph, still-pose crossfades, no leaf/feather drift, slower lighting, unchanged audio and drift, and CI media-query tests.

- **Sparse top-bar chrome**: The rationale is non-announcing controls around, not on top of, the birds. The plan has thin account/accessibility/notebook/offer/settle icons, fading chrome, no in-scene buttons, badges, tooltips, or name labels.

- **Responsive single-screen scene**: The rationale is keeping the aviary as one composed view. The plan says no pan/scroll/zoom of the scene, phone compresses horizontally, desktop widens gaps, and the sky may letterbox but never the birds.

- **Time-to-first-bird tactics**: The rationale is meeting the first-bird budget. Inline sky CSS, optional snapshot injection, code splitting, SVG parts, no required webfonts, and not waiting for WebAudio all serve first paint.

- **Procedural calls**: The rationale is distinctive living bird sound. Species motif atoms, per-bird sticky identity, mood-shaped atom selection, and seed-unique sequences avoid looped buffers and support recognizability testing.

- **Chorus mixing**: The rationale is real overlapping voices. The plan uses independent schedulers and live graphs so late clients do not start everyone at time zero and two birds are not just two file tags.

- **Listen-in mix**: The rationale is gentle focus. Gains ramp over 1.6s in both directions, ambient birds remain audible at 0.28 linear, and disengage paths are explicit.

- **Caption generation**: The rationale is captions that match what played and keep naturalist voice. The grammar emits descriptors at audio schedule time and renders fragments like a call description near the bird with AA contrast.

- **WebAudio fallback**: The rationale is accessible silence rather than recorded fallback. If AudioContext is missing, captions force on; if resume is merely gesture-blocked, the app waits silently without forcing captions.

- **Audio memory management**: The rationale is stable long-session performance. The plan preallocates grain buffers, disconnects finished nodes, avoids retained per-call arrays, and gates heap growth in CI.

- **Screen-reader narration**: The rationale is prose, not state dumps. A single polite live region updates at controlled cadence, uses server-authored naturalist prose, avoids telemetry-like labels, and treats birds as a roving tabindex list.

- **Keyboard path**: The rationale is full control without pointer dependence. Tab reaches chrome then birds, arrows move among birds, Enter toggles listen-in, Escape exits modes, and offer/settle are keyboardable.

- **Runtime captions**: The rationale is accessibility tied to actual procedural sound. Captions are generated from the grammar and forced on when WebAudio is unavailable, not a fixed map.

- **Contrast and settings**: The rationale is matter-of-fact accessibility control. Copy is AA, settings cover reduced motion and captions, narration has one cadence in v1, and privacy text is linked.

- **Performance budgets**: The rationale is preserving the quiet live scene over time. Initial JS, first-bird time, 60fps idle, heap stability, snapshot size, and tick p99 are explicit gates.

- **Aggregate-only RUM and observability**: The rationale is operational health without per-bird or relationship tracking. The plan measures timing, frame histograms, audio errors, HTTP rates, and session duration with no account or bird dimension.

- **Deliberately unmeasured data**: The rationale is privacy and anti-engagement. The plan refuses per-bird offers/listen-in/presence, trait values, mood distributions, average drift, host-identifying visit funnels, and any relationship-reconstructing metric.

- **Logging**: The rationale is preventing PII spray. Logs use account UUID only, never email or bird names, and debug sampling is off by default.

- **Build sequence**: The rationale is risk ordering. Foundations, event/schema, worker, scene, audio, interactions, notebook, visits, a11y, perf, and dogfood are sequenced so canonical state, no-vector-write tests, visual/a11y gates, and perf gates land before dogfood.

- **Bird-count ramp**: The rationale is preserving production pacing and avoiding collection framing. Dogfood does not start at seven, production gates remain unchanged, staging offsets cannot affect prod, and launch starts everyone at two.

- **Day-one instrumentation**: The rationale is operational launch visibility without engagement dashboards. It ships latency, ingest errors, magic-link consume rates, snapshot size, first-bird RUM, audio errors, visit token errors, heap, and fps, while excluding DAU and streak-like funnels.

- **Drift calibration process**: The rationale is proving slow visible change and zero negative drift. Compressed simulation checks day 7, day 21, zero-presence unchanged traits, expressiveness floor, mood cycling, and dual-device rules; human playtest asks whether birds feel different without showing numbers.

- **Feature flags**: The rationale is kill switches and tuning without violating non-goals. Visits, weather intensity, and notebook writer can be toggled; flags for personality debug, faster drift, welcome toast, and streak experiments are not allowed.

- **Testing strategy**: The rationale is making plan invariants executable. Unit, contract, property, E2E, perf CI, and a11y CI cover monotonic drift, vector write rejection, no invented presence, no spinner, listen-in ramps, settle undo, visit revoke, heap/fps, keyboard path, and live-region rate limits.
