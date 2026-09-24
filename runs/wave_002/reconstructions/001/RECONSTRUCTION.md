## System-level intent

- **Canonical state belongs to the server, with enforcement below application convention.** This shows up in the first decision: "The server is the only writer of simulation state, and the database enforces it." The plan repeats the same intent in the grants on `sim_writer`, the "single writer per aviary" lease model, append-only events, and "one canonical record" in the sync model.

- **Change must be monotonic, expressive, and long-lived.** The plan says "Drift is monotonic by construction" and later defines drift as "monotonic toward expressive." It protects this with a non-negative drive, a trigger that rejects lowered traits, daily caps, behavioral JNDs, per-bird ceilings, and a squared headroom term so heavy long-term users "keep noticing small change for years instead of saturating in months."

- **"Quieter after absence" is intentionally separated from personality drift.** The plan makes this a named design decision: "Quieter after absence is not drift." `attunement` is a separate state that affects only user-directed behavior, so traits stay monotonic while returning after two weeks can still feel "quieter than they were."

- **Presence means honest attention, not an open tab.** The plan's presence design uses a 3-signal client gate, per-minute server buckets, max-union across devices, clamps, diminishing returns, and daily caps. The rationale appears in phrases like "watching without moving is the product," "background listening would be attention the engine refuses to count," and "no double-counting and nothing lost."

- **Notice, never announce.** This phrase appears as a PRD commitment and is carried through the whole plan: no toast, snackbar, banner, spinner prompt, "click to enable sound" prompt, re-engagement notification, or catalog-like arrival UI. Greetings, candidate birds, offers, notebook observations, and matter-of-fact inline panels are the accepted surfaces.

- **Restraint is a product rule.** The plan names "Restraint (cap of 7, one screen, sparse chrome)" and backs it with the seven-bird cap, no scroll/crop layout, five fixed top-bar items, anti-feature audits, and explicit exclusions for streaks, achievements, levels, counters, visit calendars, leaderboards, notifications, and engagement surfaces.

- **Personality is latent, never a stat surface.** The plan says "Raw personality numbers never leave the simulation zone" except for account export, and "Personality never shown." This intent reappears in the expression compiler, snapshot schema, trait-term linter, support tooling, export limits, and "no trivial extension that reads traits."

- **Privacy is an architectural rule, not policy.** The plan uses that exact phrase for the Zone A/B/C topology. It also says telemetry has no database route or credentials, RUM is cookieless and identifier-free, there are no product analytics SDKs, and production behavior cannot tune drift because "the privacy rule forbids population-level analysis."

- **The product voice is authored, sparse, naturalist, and controlled.** The plan rejects an LLM for notebook, narration, and captions because a grammar gives "voice control, sparsity control, and output we can lint." It reinforces this with writer ownership, lowercase naturalist strings, no second person, no trait names, no gamified words, and a separate matter-of-fact system-copy registry.

- **Accessibility is a designed surface, not a fallback.** The plan says accessibility "ships in v1 as a designed surface" and that narration, captions, and reduced motion each have "their own design and owner." Reduced motion is "a launch gate, not a derived fallback," and accessibility sign-off is a launch blocker.

- **First contact should feel alive, not loaded.** The plan requires "First frame without a spinner," "first frame is mid-action," and "no entry animation." The quiet field exists to hold the screen without loading theater, and birds fade in "already in motion" so the aviary reads as resolving rather than starting.

- **Procedural and deterministic systems preserve continuity and auditability.** The tick is a deterministic function, RNG streams are keyed so adding a subsystem does not shift others, math is vendored to avoid engine/version dependence, audio is synthesized, and the prose system is grammar-based rather than model-generated.

- **Graceful degradation should preserve the aviary's spell.** The quiet field, local ambient continuation, silent-with-captions fallback, reduced-motion rendering, and quality governor all degrade without replacing the aviary with loading chrome. The quality governor specifically "never lowers bird motion frequency, the motion of the aliveness surface itself."

- **Changes after launch move forward and blend, never rewrite the past.** The plan uses `params_version`, forward-only engine constants, no retroactive recomputation, 7-day expression/art blends, species generations for new birds, no import, and permanent bird identity. It says engine changes "never recomputes the past" and existing birds are "never removed."

## Per-feature whys

### 0. Orientation and 1. Scope

- **One aviary per account.** Why: the plan uses `aviaries.account_id UNIQUE` to keep out shared or multi-aviary accounts, payments, discovery, leaderboards, and cross-account read paths; this supports restraint and the privacy boundary.

- **Two starter birds.** Why: starters are two diurnal species with separated registers and silhouettes so "the first encounter must be legible in daylight."

- **Cap of 7 birds.** Why: the plan ties the cap to "Restraint," one-screen layout, no crop/no scroll guarantees, and performance ramping; at 7 birds, candidates stop and "nothing already given is taken away."

- **Six species.** NOT RECOVERABLE FROM PLAN

- **Stable bird identities.** Why: bird UUIDs, `voice_params`, `look_params`, and `species_asset_gen` are permanent so identity continuity survives art, voice, and engine changes.

- **Renameable birds.** NOT RECOVERABLE FROM PLAN

- **Server-side 60 s tick.** Why: the plan says the tick runs "whether or not any client is connected" in simulated time, while hot and dormant schedules keep fleet cost proportional to activity.

- **Personality drift.** Why: drift makes long-term attention become visible, but the plan constrains it to be monotonic, capped, low-pass-filtered, and slow enough that "a single session is never visible."

- **Attunement.** Why: it reconciles "never drifts down on neglect" with a return that is "quieter than they were," without touching traits, wariness, plumage, or unobserved call rate.

- **Mood.** Why: mood gives canonical, persistent session-to-session variation, with a "daily-ish reset" from circadian dawn rather than a tab opening.

- **Day/night in the account's timezone.** Why: the plan says "Mood and light must agree"; a canonical timezone avoids split-brain time of day across devices.

- **Ambient weather.** Why: weather shapes expression, mood, call rates, and visible reactions while remaining deterministic and never becoming a trait change.

- **Bird-to-bird social behavior.** Why: affinity, proximity, responses, and social dynamics are part of keeping the aviary moving over time and mitigating the risk that traits saturate.

- **Call bouts, choruses, and alarm contagion.** Why: they make bird behavior relational and specific: responses create call-and-response, choruses get leaders and joiners, and visible startle causes avoid tying alarm to the user's presence or absence.

- **New birds by aviary age.** Why: arrivals depend only on aviary age, not user behavior, so new birds are not an engagement reward; arrival ages can be stretched only forward.

- **Return-greeting.** Why: the greeting engine is "the whole welcome surface" and lets a returning tab greet within 0.6-2.0 s with no network round trip.

- **Presence accounting.** Why: the plan wants honest attention, so it credits only gated presence, clamps physical limits, takes a union across devices, and applies diminishing daily returns.

- **Listen-in.** Why: listen-in brings one bird forward while keeping others audible, supports focused captions, and gives the focused bird warmth/vocal credit only where it intersects with real presence.

- **Offers.** Why: offers stay "a gesture, not a tool," are resolved canonically on the server, and store the recorded reaction so "the device saw exactly what became canonical."

- **Settle with 5 s undo.** Why: settle is local to the device, sends its event only after the undo window, avoids server compensation on undo, and gives only a "small mood-quieting signal."

- **Field notebook.** Why: the observer makes drift observable without numbers through sparse naturalist observations, while never referencing the user, absence, visit counts, trait names, or trait numbers.

- **Single responsive scene.** Why: the plan's restraint principle requires one screen with no scroll and no crop; the layout solver proves birds and flight arcs stay inside the safe area.

- **Three perch zones.** NOT RECOVERABLE FROM PLAN

- **Idle micro-motion, flight transitions, weather, and lighting.** Why: these make the first frame and ongoing scene feel alive, with birds already mid-preen, mid-scan, or mid-call rather than entering through a loading sequence.

- **Top bar that fades.** Why: sparse chrome keeps the aviary dominant; the bar fades to 8% opacity and returns on intent, with a setting to disable the fade for accessibility.

- **Quiet field.** Why: if the snapshot is late, the quiet field avoids spinner/loading UI; when birds appear already in motion, it reads as "the aviary resolving, not as an entry sequence."

- **Empty-aviary state with fly-in.** NOT RECOVERABLE FROM PLAN

- **Procedural calls.** Why: the plan bans recorded audio and makes calls procedural so captions can match generated `CallSpec` objects and CI can fail any audio asset in `dist/`.

- **Chorus mixing and listen-in mix.** Why: chorus turn-taking follows "real acoustic-niche behavior," and listen-in reads as focus plus distance rather than muting the rest of the aviary.

- **Procedural ambient bed.** Why: wind, rain, and a quiet night texture mean "night is never dead air" without recorded assets.

- **Silence-with-captions fallback.** Why: when WebAudio is unavailable, calls are still generated for captions and beak motion, and captions default on.

- **Screen-reader narration.** Why: accessibility is a designed surface, and narration uses naturalist prose to cover scene, focal bird, and salient changes without assertive interruption.

- **Call captions.** Why: captions are generated from the same `CallSpec` as the audio, so they describe "exactly what was played."

- **Reduced-motion mode.** Why: vestibular accessibility is treated as its own designed rendering with pose sheets, cross-fades, rain wash, unchanged captions/narration/audio, and launch-gate review.

- **Full keyboard operation.** Why: keyboard access is part of the accessibility launch gate, with stable focus order so moving birds do not reorder the list.

- **WCAG AA contrast and forced-colors support.** Why: the plan makes contrast, forced-colors chrome, caption plates, and matrix tests launch requirements for accessibility.

- **Magic-link sign-in.** Why: account existence is never revealed, scanners cannot consume tokens, and tracking rewrites are disabled because they break single-use links and leak behavior.

- **Per-device sessions with revocation.** Why: the session list gives device label and sign-in date, enough to spot an unknown device, while avoiding "last active" because it edges toward a visit-frequency surface.

- **Verified email change.** Why: the old email keeps working until the new one verifies, and the old address gets a matter-of-fact notice after the switch.

- **JSON export delivered by email link.** Why: raw traits are included only for portability/data-access obligations, never previewed in the product, rate-limited, and not importable so export does not become a stat loop.

- **Soft delete then hard delete.** Why: soft delete gives "I changed my mind" recovery while hard delete removes Zone A/B data, destroys the DEK, purges caches/exports, and re-applies tombstones on restore.

- **Visits.** Why: visits are the only social surface, read-only and ambient; visitors cannot write interaction events, see the host notebook, use offers, or access host-private fields.

- **Visit log, unused-invite expiry, revocation, and opt-in visit notice.** Why: the plan honors the PRD while minimizing a non-user's PII, keeps notices opt-in, and makes revoked/deleted visits return the same unavailable surface.

- **Aggregate-only telemetry.** Why: telemetry has no identifiers or route to simulation data, so operations can be observed without per-account or per-bird behavior.

- **Synthetic performance checks.** Why: synthetic accounts let the team measure first-bird, frame timing, audio init, listen-in, offer, and settle without production behavior analysis.

- **Tick-latency alarms and integrity monitors.** Why: they catch the "silent worst case" of reset or lost personality, event-cursor regressions, lagging aviaries, and monotonic-trigger violations.

### 1.2 Out of v1 and 1.3 decisions

- **Native apps excluded.** Why: browser-only protocols keep out push tokens and offline-first sync.

- **Gamification excluded.** Why: no per-day visit data, counters, streaks, achievements, levels, visit calendars, or copy lexicon are allowed because they would violate restraint and visit-frequency boundaries.

- **Tamagotchi mechanics excluded.** Why: the schema has no hunger, health, or happiness-decay fields, and drift cannot go negative.

- **Social network surfaces excluded.** Why: visits are the only social table family, with no profile, follow, comment, or invitation free text.

- **Notifications and re-engagement excluded.** Why: the mailer accepts only allowlisted transactional templates; anything else is rejected in code, preserving "notice, never announce."

- **Payments, shared accounts, multi-aviary accounts, custom scenes, discovery, and leaderboards excluded.** Why: unique account-to-aviary mapping, no cross-account read path, and no cross-account aggregates keep those surfaces out.

- **Recorded audio excluded.** Why: the plan commits to procedural calls and build checks that fail any audio MIME, magic bytes, or data-URI audio.

- **Trait stats in any view excluded.** Why: no endpoint returns traits except export, and support tooling shows only integrity pass/fail so personality is never a stats surface.

- **Importing an export excluded.** Why: import would be a "last-write-wins path for personality."

- **Releasing or rehoming a bird excluded.** Why: birds leave only on hard delete because release/rehoming would conflict with identity continuity.

- **Settle as a fifth top-bar item.** Why: two later, more specific sections require settle in the top bar, so the four-icon list is treated as earlier chrome inventory.

- **Mute in accessibility settings and audible-presence bonus.** Why: the brief names mute as behavior birds respond to; monotonicity means muting can only withhold a vocal-frequency bonus, never subtract.

- **AudioContext created at boot without an enable-sound prompt.** Why: a prompt "would announce"; returning users may get autoplay, and others resume on first gesture with a fade.

- **Presence activity window set initially to 5 min.** Why: the plan says to "lean long" because "watching without moving is the product."

- **Modern activity signals.** Why: `keypress` is deprecated and touch rarely emits `pointermove`, so `pointermove`, `pointerdown`, and `keydown` preserve the same intent in modern APIs.

- **Absence length across devices.** Why: "The laptop five minutes ago means the phone gets a glance, not a re-orientation."

- **Settle scope local to device.** Why: "Settling on the laptop should not put the phone to evening."

- **Hidden-tab audio fade and suspend.** Why: there is "nothing to see," battery matters, hidden time is never presence, and background listening would be attention the engine refuses to count.

- **Visitor-local accessibility settings.** Why: "Accessibility must reach visitors too," while the notebook belongs to the host.

- **Claimed-visit lifetime and visit-log retention.** Why: active-until-revoked honors the PRD, and rolling retention/purges minimize a non-user's PII.

- **No personal message in an invitation.** Why: fixed matter-of-fact text prevents "chat-by-invite and spam abuse."

- **Canonical account timezone.** Why: "Mood and light must agree," and a natural "jet lag" beats split-brain time of day.

- **No nocturnal species as starter.** Why: the first encounter must be legible in daylight, while nighttime still has sleep shuffles, murmurs, and night texture.

- **Candidate bird visits and adoption card.** Why: the plan wants "Discovery by noticing"; there is no toast, and arrivals depend only on aviary age.

- **Session list without "last active."** Why: "Last active" edges toward a visit-frequency surface, while sign-in date is enough to spot an unknown device.

- **English-only localization.** Why: the grammar would have to be re-authored per language.

- **Fixed day curve with no latitude or seasons.** Why: "We never ask for location."

- **Reduced-motion preference stored at account level.** Why: "A vestibular need follows the person across devices."

- **Adoption entry in a new notebook.** Why: the notebook is never empty, and the first entry is an observation.

- **Offer placement without placement UI.** Why: this keeps offers "a gesture, not a tool" while keyboard users get targeted placement for free.

- **Bird-count telemetry bucket.** Why: coarse buckets with k-anonymity are a privacy-reviewed operational exception needed "to ramp bird counts safely."

### 2. Architecture, 3. Data model, and 4. API surface

- **Zone A identity, Zone B simulation, Zone C telemetry.** Why: this is the privacy "architectural rule, not policy"; Zone C has no peering or credentials for the database subnet.

- **Only five deployables.** Why: keeping `edge`, `api`, `sim-worker`, `jobs`, and `web` as the deployable set keeps PII boundary crossings few.

- **Transactional email with open and click tracking disabled.** Why: click-tracking rewrites links, breaks single-use magic links, and leaks behavior.

- **Email blind index normalization without plus-address collapse.** Why: plus-addresses are not collapsed "because the user owns that distinction."

- **Versioned pepper with dual lookup.** Why: the dual lookup exists for a transition window during rotation.

- **Database grants for simulation state.** Why: "Clients never write personality" is a grant, not a convention; API can insert events and rename birds, but cannot update simulation tables.

- **Append-only notebook entries.** Why: notebook rows are simulation side effects and are protected from update/delete except by deletion work.

- **Trigger rejecting an 8th active bird.** Why: the seven-bird cap is enforced by both engine and database, not only UI.

- **Raw interaction event retention for 30 days.** Why: raw events are needed for recovery replay and observer windows, then partitions are dropped.

- **Secure session cookie plus CSRF header.** Why: mutations need both the opaque session cookie and `X-Aviary-CSRF` to harden authenticated writes.

- **Edge token with revocation set.** Why: the edge can serve fast snapshots while checking revoked sessions within propagation of at most 60 s.

- **Emails never in URLs, query strings, or logs.** Why: email belongs only in the identity zone and mailer/settings/visit-log views.

- **Typed error codes mapped to system copy.** Why: system prose stays matter-of-fact and centralized; the server never sends user-facing prose for errors.

- **Magic-link request always returns `202`.** Why: account existence is never revealed.

- **Verify landing with auto-POST only in the same browser.** Why: link scanners and cross-browser opens cannot consume the token without a "Continue signing in" action.

- **Per-aviary event sequence equals commit order.** Why: the counter row lock removes the "classic cursor-skip hazard of global identity columns."

- **Visitor snapshot bird handles.** NOT RECOVERABLE FROM PLAN

- **Visitor claim button.** Why: the landing page requires "View aviary" so scanners do not claim the invite.

- **Visitor heartbeat writes only `visit_log`.** Why: visitor activity never reaches `interaction_events`, so visits stay read-only and do not affect simulation.

- **Persistent inline system messages.** Why: errors remain matter-of-fact and are never toasts or timer-dismissed announcements.

### 5. Simulation engine

- **Hot and dormant schedules.** Why: the tick advances persisted canonical state and scales fleet cost with activity while hot and dormant schedules produce byte-identical results.

- **RNG streams by subsystem.** Why: adding a subsystem should not shift the others.

- **Vendored pure-TS math.** Why: results must not depend on engine or version.

- **Golden fixtures and `params_version`.** Why: any byte change needs a reviewed diff and a forward-only constants version.

- **Presence max across devices.** Why: max-union approximates attention without letting two open devices double-count.

- **Diminishing daily returns and cap.** Why: heavy attention is bounded so no single day contributes more than a quarter JND and one session is never visible.

- **Audible share.** Why: audible presence can add a small vocal-frequency bonus while muting only withholds the bonus.

- **Listen-in credit intersected with presence.** Why: listen-in counts only when the user is actually present in the same minute.

- **Offer cooldowns and accepted-offer caps.** Why: birds can notice over-cap offers, but drift credit is bounded against offer spam.

- **Settle and reengage feeding mood only.** Why: settle is a quieting signal, not personality drift.

- **Low-pass drive that keeps drifting after departure.** Why: it implements "personality drifts during the user's absence based on inputs from before they left" without inventing anything at return.

- **Squared headroom term.** Why: traits slow as they near ceilings so long-term users still notice small changes for years.

- **Per-bird trait ceilings.** Why: birds remain distinct even after years of attention.

- **Behavioral JNDs.** Why: traits are measured through expression and choreography statistics, not displayed as numbers.

- **Attunement limited to greeting, approach, and gaze.** Why: the plan explicitly prevents attunement from affecting wary/ease axes, plumage, unobserved calls, perch choice outside presence, or traits.

- **Mood dawn reset.** Why: simulated-time circadian dawn resets mood toward personality baseline; opening the tab never resets mood.

- **Weather as expression and affect, not trait.** Why: rain and wind shape call rate, arousal, and wary impulses without changing personality.

- **No storms and no snow.** NOT RECOVERABLE FROM PLAN

- **Startles with visible subtle causes.** Why: alarm comes from a branch, cloud shadow, or similar cause and is "never tied to presence or absence."

- **Canonical perch slot assignment.** Why: every device shows the same bird on the same slot.

- **Mood-shaped activities.** Why: content birds preen, wary birds scan/back up, curious birds tilt, and drowsy birds fluff, making mood readable.

- **Affinity.** Why: affinity grows through co-perching and responses, shapes proximity and response probability, and is explicitly not a personality trait.

- **Offer resolver with recorded reaction.** Why: synchronous deterministic resolution makes the client reaction and canonical tick outcome exactly match.

- **Song-fragment library as note data.** Why: it is played through the synthesizer and "never a recording."

- **Notebook observer excluding presence and absence.** Why: the notebook cannot imply visit frequency or user behavior.

- **Notebook sparsity budget and natural writing moments.** Why: observations stay sparse, dated locally, and written at mid-morning or evening rather than every event.

- **Notebook forbidden content.** Why: entries never reference the user, absence duration, visit counts, traits, or numbers.

- **Adoption bootstrap freezing voice and look parameters.** Why: pitch, signature motif, timbre, markings, and palette endpoints preserve each bird's identity.

- **Candidate "let it go."** NOT RECOVERABLE FROM PLAN

- **Expression compiler quantization and mixing.** Why: no single output is an affine readout of a trait, preventing a stats surface or trivial trait-reading extension.

- **Expression mapping blend over 7 days.** Why: no bird should visibly "snap" after a deploy.

### 6. Sync model

- **One canonical record.** Why: every device and visitor reads the same snapshot, and no client holds state another client needs.

- **Single writer with fenced leases.** Why: split-brain cannot commit, and state, cursor, personality, and notebook side effects commit atomically.

- **No retroactive recomputation.** NOT RECOVERABLE FROM PLAN

- **Pending-offer overlay in snapshot reads.** Why: a second device sees an offer on its next pull rather than waiting a minute.

- **Snapshot pull jitter and `304` responses.** Why: jitter spreads load and `304` makes unchanged plans cheap.

- **Client reconciliation without visible teleports.** Why: when a plan changes, birds wait for a natural boundary or ease into the target so there are "No teleports while visible."

- **Ambient continuation during outage or offline state.** Why: the scene should not freeze or error unless a user action actually needs the server.

- **Outbox with idempotency keys.** Why: events can flush on reconnect or `pagehide`, and retries are safe.

### 7. Frontend rendering pipeline

- **Boot path with inline snapshot and Early Hints.** Why: the first bird target is under 500 ms, with a tiny core renderer and no spinner.

- **Quiet-field slow path.** Why: slow snapshots still show sky and subtle motion, then birds fade in already moving so it is not an entry animation.

- **Scene layers and cached canvases.** Why: cached layers allow Canvas2D to meet frame budgets while birds and offers redraw every frame.

- **Quality governor degradation order.** Why: it sheds particles, parallax, DPR, and feather detail before touching bird motion, because bird motion is the "aliveness surface itself."

- **Layout solver safe area.** Why: it guarantees no crop, no scroll, no birds under the top bar, and no unsafe flight arcs across viewport sizes.

- **Resize/orientation easing.** Why: birds keep zone and slot identity and ease to new coordinates rather than jumping.

- **Procedural bird rig and idle motion.** Why: breathing, saccades, shuffles, tail flicks, fluffing, blink, beak sync, and activity clips make each bird feel alive without looping.

- **Motion frequency ceiling.** Why: no oscillation above 3 Hz except saccades/wingbeats, avoiding strobing.

- **Second-scale noticing without rings, badges, or glow.** Why: birds can register attention with gaze while avoiding UI marks inside the scene.

- **Allocation discipline and pools.** Why: the per-frame path allocates nothing, supporting the "no growth over 30 minutes" budget.

- **Greeting choreography on the client.** Why: greetings happen within 0.6-2.0 s without a network round trip, vary by absence/intensity, and are recorded afterward for notebook use.

- **Settle choreography.** Why: lighting and audio move toward evening, birds respond softly, and undo reverses locally before any event is sent.

- **Offer anticipatory beat.** Why: birds notice the item while the request is in flight, preserving immediacy before the canonical reaction arrives.

- **Panels that keep scene and audio running behind them.** NOT RECOVERABLE FROM PLAN

- **Adoption card with no catalog, stats, or adoption dates.** Why: the card stays in naturalist voice and avoids stats/counter surfaces.

- **Reduced-motion pose and cross-fade design.** Why: motion-sensitive users get an authored surface with removed parallax, leaves, streaks, and wingbeats while keeping the product's content.

- **Visibility lifecycle handling.** Why: hidden tabs stop rAF, audio, and heartbeats because hidden time is never presence and battery matters.

- **Memory lifecycle limits.** Why: fixed pools, virtualized lists, pooled caption nodes, one AudioContext, and at most two snapshots keep heap, DOM, AudioNodes, and canvases constant.

### 8. Audio pipeline

- **Per-bird immutable voice signature.** Why: each bird stays recognizable across mood and drift; traits can change tempo, loudness, ornaments, motifs, bout length/rate, and alarm use, but not register, signature motif, or timbre.

- **`CallSpec` as one source of truth.** Why: the same object drives synthesis and captioning, so captions match the played call.

- **Chorus mixing with turn-taking.** Why: deferring extra bouts creates "real acoustic-niche behavior" rather than sample-locked overlap.

- **Listen-in mix with slow ramps.** Why: +4 dB focus and softened other birds read as "further away"; the ramp is "slow, not a switch," and others are never silent.

- **Ambient bed.** Why: the low procedural bed provides air, leaves, rain, and quiet night texture so the scene is not dead air.

- **Autoplay fallback and silent mode.** Why: no prompt is ever shown; audio resumes on gesture when possible, and permanent silent mode keeps captions and beak motion synchronized.

- **No recorded audio enforcement.** Why: procedural sound is enforced by CI scanning `dist/`, note-data song fragments, and code-only worklets.

### 9. Accessibility surfaces

- **Scene semantics and transparent bird buttons.** Why: the canvas scene becomes operable as one "aviary" group with roving bird controls and 44 x 44 px hit boxes.

- **Captions marked `aria-hidden`.** Why: narration already covers audio for screen-reader users, so captions avoid double speech.

- **Narration cadence and queue merging.** Why: narration stays polite, sparse, and rate-limited, never assertive live-region spam.

- **Caption placement and plates.** Why: captions must remain readable against dynamic lighting/weather, avoid birds/top band/each other, and pass 4.5:1 contrast tests.

- **Frozen keyboard focus order inside the scene.** Why: moving birds do not reorder the list while focus remains in the scene.

- **Focus indicator as the one sanctioned scene mark.** Why: keyboard users need a visible indicator, while the scene otherwise avoids rings, badges, and glow.

- **Presence equity for assistive technology.** Why: screen-reader users in browse mode may be under-credited, so the plan tests parity and would widen the window rather than loosen the three-signal conjunction.

- **Accessibility test matrix and gate.** Why: screen-reader, reduced-motion, keyboard, contrast, and axe checks are launch blockers.

### 10. Voice and content system

- **Typed grammar templates.** Why: templates declare inputs so they cannot reference data they were not given, such as presence.

- **Writer ownership and sample review.** Why: a staff writer controls voice, reviews generated samples, and prevents drift into generic or gamified phrasing.

- **Naturalist-string linter.** Why: lowercase, no second person, no digits, banned gamification/visit words, and no trait names keep notebook, narration, and captions in the intended register.

- **Separate system-copy registry.** Why: system copy uses the "opposite register": sentence case, no naturalist vocabulary, and centralized error strings.

### 11. Privacy, security, and account lifecycle

- **Email ciphertext and per-account DEK.** Why: email exists only in encrypted form except mailer/settings/visit-log views, and hard delete can crypto-shred lingering backup ciphertext.

- **Allowlisted logging and self-hosted error tracking.** Why: request bodies and event payloads are never logged, unknown fields are dropped, and no user context or click breadcrumbs leave the product.

- **PII canary.** Why: a nightly synthetic account finds any email or blind-index leak into Zone C and pages on-call.

- **Cookieless RUM boundary.** Why: RUM measures only aggregate histograms/counters, drops IP after coarse country, and carries no user, session, or aviary identifiers.

- **Deliberately unmeasured behavior.** Why: the privacy rule forbids drift, mood, notebook, listen-in, offer, greeting, retention, and visitor behavior analysis in telemetry; tuning uses simulation and dogfood.

- **Auth hardening with hashed single-use tokens and opaque sessions.** Why: tokens cannot be replayed, scanners are contained, and sessions are revocable with browser/OS-only labels.

- **Bird-name validation.** Why: visitors see host-chosen names in narration, so names must be sanitized and rendered as text only.

- **Export and deletion mechanics.** Why: export provides data access without product stats, and deletion destroys identity/simulation rows, DEKs, caches, pending offers, exports, and restored tombstoned accounts.

### 12. Performance, 13. Testing, 14. Rollout, and 15-16 risks/open items

- **Performance budgets.** Why: the internal targets make "first bird" faster than the PRD ceiling and keep long sessions at 60 fps with no memory growth.

- **Synthetic measurements.** Why: synthetic accounts and browsers validate performance and core flows while avoiding production behavior telemetry.

- **Integrity counters.** Why: monotonic violations, personality reset suspects, cursor regressions, missing personality rows, and lagging aviaries catch severe silent failures.

- **Testing and anti-feature audit.** Why: property tests, calibration, chaos, contract tests, visual regression, audio panels, and greps enforce the product promises and prevent "announcement creep."

- **Dogfood timing.** Why: dogfood starts at least four weeks before beta because the 3-week visible-drift target can only be validated in real time.

- **Bird-count ramping.** Why: production will not naturally reach 5-7 birds until later, so staff-only time-travel aviaries test them first; if performance regresses, future arrivals slow and existing birds remain.

- **Post-launch engine and art change policy.** Why: constants are forward-only, art/expression changes blend, species updates create new generations, and the past is never recomputed.

- **Engagement-surface proposals.** Why: any streak, badge, notification, or "harmless" counter is "out of scope by charter and needs no debate."
