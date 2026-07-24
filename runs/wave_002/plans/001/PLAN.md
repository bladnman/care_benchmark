# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In v1
- Web-only aviary: one horizontal scene per account, rendered in modern browsers (last 2 major versions of Chrome, Safari, Firefox, Edge).
- Two starter birds per new account, cap of seven; additional birds unlock by aviary age only.
- Full bird engine: hidden personality vectors, mood system, procedural calls, monotonic-toward-expressive drift.
- Interactions: presence accounting (3-signal conjunction), return-greeting, listen-in, offer (seed / song fragment / still pool, per-bird cooldown), settle (with 5s undo), field notebook (auto-generated, read-only, sparse).
- Single-user accounts with email magic-link auth, per-device revocable sessions, email change with verification.
- Server-side simulation tick (~1/min) as the only writer of personality state; multi-device sync as a property of that architecture.
- Visit invitations: per-invite opt-in, read-only ambient, revocable, 30-day expiry, silent visit log, off-by-default visit notifications.
- Accessibility surfaces shipping with v1: screen-reader naturalist narration, designed reduced-motion mode, procedural call captioning, keyboard navigation, WCAG AA chrome contrast.
- Account export (JSON, emailed download link) and soft-delete (30 days) then hard-delete.

### Explicitly out (per non_goals.md)
- No native apps, no gamification of any kind (no streaks, counters, badges, visit calendars), no Tamagotchi mechanics (no death, hunger, distress, decaying meters), no social-network surfaces (profiles, follows, feeds, discovery, leaderboards, co-presence, chat, avatars, comments), no notifications pushed at the user, no recorded audio, no exposing personality vectors numerically anywhere.

### Defensible calls on ambiguities
- Mood enum finalized as: `wary, content, curious, drowsy, alert, settled` (settled covers night/settle-gesture states).
- Personality traits normalized to [0.0, 1.0], seed values drawn from a per-species prior with small per-bird jitter.
- Presence activity window: 4 minutes (leaning long per PRD guidance).
- Offer cooldown: 5 minutes per bird.
- Snapshot keepalive cadence: 30s while visible; pull on visibilitychange and on >5s render gaps.

## 2. Architecture

### Service shape
Three deployable units plus a client:

1. **Web client** — static SPA served from CDN. Rendering, WebAudio synthesis, presence detection, interaction capture. Owns no canonical state.
2. **API service** — stateless HTTP: auth (magic link issue/consume), session management, snapshot reads, interaction-event ingest, notebook reads, visit-invite lifecycle, account settings/export/deletion.
3. **Simulation service** — the tick. Cron-driven worker (or queue-partitioned workers at scale) that, per account, reads unprocessed interaction events, advances moods and drift, emits canonical state and occasional notebook entries, and checkpoint-writes to the store.
4. **Datastore** — single relational store (Postgres) for accounts, birds, sessions, invites, notebook entries, event log (append-only table), canonical state checkpoints. The simulation DB is never read by analytics pipelines (privacy boundary enforced at infra level: no warehouse connection, separate credentials).

### Client/server split
- Server owns: personality vectors, moods, canonical positions, drift, notebook generation, visit auth, presence-time accounting (from client-submitted presence pings, validated server-side).
- Client owns: rendering, interpolation between snapshots, call synthesis from server-delivered call-grammar parameters + PRNG seeds, ambient ornaments (leaves/feathers — pure client, no state), caption text generation from the same call parameters it synthesizes.

### Render pipeline boundary
The client receives a **snapshot**: per bird — id, species, perch zone + fine position, mood, current call schedule (next-call timestamps + motif params), personality-derived render hints (plumage saturation scalar, boldness → perch bias already baked into position). The client never derives behavior from raw trait values; the server pre-resolves behavior into snapshot fields. This keeps personality vectors invisible on the wire (privacy and the "never expose numbers" rule — render hints are coarsened, e.g. plumage bucketed into 8 levels).

## 3. Data model

**Account**: `id` (synthetic UUID, the only identifier used in logs/telemetry/sharding), `email_encrypted`, `created_at`, `deletion_pending_at nullable`, settings JSON (reduced-motion opt-in, captions, visit-notify toggle, audio on/off).

**Session**: `id`, `account_id`, `device_label`, `created_at`, `revoked_at nullable`.

**Bird**: `id` (stable UUID, never regenerated), `account_id`, `species_id`, `name`, `adopted_at`, `personality_vector` (boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity — floats [0,1]), `mood`, `mood_entered_at`, `perch_zone`, position fields. Server-only write access.

**InteractionEvent** (append-only): `id`, `account_id`, `bird_id nullable`, `type` (presence_ping, listen_in_start/end, offer, settle, settle_undo), `payload` JSON, `client_ts`, `server_ts`, `consumed_by_tick_at nullable`. Never aggregated cross-account.

**NotebookEntry**: `id`, `account_id`, `bird_id nullable`, `prose`, `created_at`, `generation_key` (dedup key so the same observation isn't written twice).

**VisitInvite**: `id`, `host_account_id`, `visitor_email_encrypted`, `token_hash`, `created_at`, `expires_at` (30d), `revoked_at nullable`, `first_used_at nullable`.

**VisitLogEntry**: `id`, `invite_id`, `started_at`, `approx_duration_s`.

**CanonicalStateCheckpoint** (optional optimization): latest snapshot per account so snapshot reads don't recompute; tick rewrites it each pass.

## 4. API surface

All authenticated by per-device session token (httpOnly cookie or Authorization header). JSON over HTTPS.

- `POST /auth/magic-link` — {email} → sends link (15-min expiry, single-use, per-email rate limit).
- `GET /auth/consume?token=` → issues session token; invalidates link.
- `GET /aviary/snapshot` → current canonical snapshot (small, KBs; also inlined into initial HTML payload for first-bird <500ms).
- `POST /events` — batch ingest of interaction events (presence pings, listen-in start/end, offer, settle/undo). Server validates, appends, returns 202. Never accepts personality fields.
- `GET /notebook?before=` — paginated entries.
- `POST /birds/{id}/rename` — name only.
- `POST /adoptions/respond` — accept/decline an age-unlocked species offer.
- `POST /visits/invites` — create invite (email); `DELETE /visits/invites/{id}` — revoke; `GET /visits/log` — visit log.
- `GET /visit/{token}/snapshot` — visitor snapshot read (render-only; rejects event ingest; logs visit duration silently).
- Account: `GET/PUT /account/settings`, `POST /account/email-change`, `POST /account/export` (emails link), `POST /account/delete`, `POST /account/recover`, `GET /account/sessions`, `DELETE /account/sessions/{id}`.

## 5. Simulation engine design

### Tick
Runs every ~60s across all accounts (sharded by account UUID when scale demands). Per account:
1. Read events since last consumed watermark, in order.
2. Fold presence pings into presence-time per session-window; attribute listen-in/offer events to birds.
3. Update **drift**: low-pass filter `trait += k * signal * (1 - trait)`, k tuned so instruments see change at ~1 week of regular visits and users feel it at ~3 weeks. Monotonic: signals only push up; absence applies zero (never negative) delta. Drift inputs weighted: presence-time (dominant, aviary-wide), listen-in (→ social warmth, vocal frequency, per bird), offers (accepted → curiosity; offered-near → boldness), settle (no directional drift, closes presence window cleanly).
4. Update **mood**: per-bird state machine over {wary, content, curious, drowsy, alert, settled}. Transition inputs: recent-session interactions, local time-of-day bands (drowsy near dusk, alert early morning), ambient weather events (rain dampens vocal frequency aviary-wide; wind → alert/wary split by boldness), personality modulation (high boldness suppresses wary). Mood persists across sessions; tick advances mood timers during absence (e.g., overnight drowsy → settled → morning alert/content).
5. Bird-to-bird coupling: call events from bird A probabilistically prompt response scheduling for bird B scaled by B's vocal frequency and warmth; wary spreads with a short decay; chorus emerges when ≥2 high-vocal-frequency birds have overlapping call windows.
6. Schedule next calls per bird (timestamps + motif parameters + seed), choose perch positions from mood×personality, advance weather state machine (rare rain a few times/week, occasional wind).
7. Possibly emit a **notebook entry** (see below).
8. Write canonical state + checkpoint. Mark events consumed.

Tick p99 latency alarm at 5s. Tick is idempotent per account (watermark + unique event ids) so retries are safe.

### Call grammar runtime
Per species: motif library (small parametric tone envelopes: pitch contour, trill rate, duration, harmonic profile). Runtime call = motif choice (weighted by mood) + parameter jitter (seeded RNG) + personality shaping (vocal frequency → rate; warmth → response likelihood). Call *signature* stability: each bird has a fixed seed-derived timbre offset so it stays recognizable across mood/drift. Server schedules; client synthesizes; caption text derived from the same motif+parameters.

### Return-greeting selection
On session start (first presence ping after absence), tick-side (or snapshot-time) logic picks the greeter: weighted by boldness × mood-availability; absence length bucketed (minutes/hours/days) selects greeting intensity (glance / two-note call / approach + longer call). If two birds qualify, stagger by random 0.5–2s offset. Parameters shipped in snapshot; client renders procedurally varied execution — never a canned clip.

### Notebook generation
Event-driven candidate detector (greeting-order novelty, long quiet stretches, unusual perches, chorus events, offer firsts) feeds a prose templating layer with strong variation (phrase banks, clause composition, day-part and weather lexicon). Sparsity governor: max ~1 entry per 2–3 days baseline; noteworthy events can add more but with per-week cap. Entries observe the aviary, never the user's behavior (no "you visited" phrasing, ever).

## 6. Sync model

Single canonical state per account, server-only writes. Multi-device coherence is reading the same record; there is nothing to merge.

- Clients pull snapshots: on load, on visibilitychange → visible, on long frame gaps, 30s keepalive. Interpolate positions between snapshots (no teleporting).
- Clients append events only. Personality changes are additive server-authored deltas applied in event-log order — no last-write-wins anywhere.
- Conflict surface is minimal by design: sessions can be revoked, magic links replayed, tokens expired — all handled with matter-of-fact error copy and re-auth. Two devices interacting simultaneously is safe: events are ordered server-side; the tick serializes consumption per account.
- Visit revocation is enforced at snapshot-pull time: next pull returns the revoked surface.

## 7. Frontend rendering pipeline

- **Scene**: single horizontal canvas (WebGL or 2D canvas chosen by capability probe; SVG/DOM fallback only for unsupported → unsupported-browser surface per PRD, we do not maintain old-browser paths). Three parallax planes: background foliage/sky, middle (birds + perches), occasional foreground branch. All birds always in frame; responsive compression/expansion of perch spacing with fixed aspect envelope.
- **First frame**: snapshot inlined with HTML payload at CDN edge; birds placed mid-action with phase-offset idle loops; no spinner, no entry animation. Slow-connection loading state is the "quiet field" (soft sky, faint motion cues). Budget: first bird visible <500ms on mid-tier mobile over 4G; initial JS <2MB gzipped, aggressive code-splitting for settings/visit/notebook surfaces.
- **Idle micro-motion**: mood-keyed animation state machines (preen, scan, head-tilt, weight-shuffle) with personality-modulated frequency; continuous, phase-randomized per bird so nothing syncs up. 60fps sustained on 5-year-old laptop; render loop halts when hidden (simulation continues server-side).
- **Ambient ornaments**: leaves/feathers drift client-side at slow random cadence; no simulation state.
- **Day/night**: palette keyframes driven by local time; gradual transitions; night state with one nocturnal species active. Weather overlays: subtle rain/wind layers keyed from snapshot.
- **Top bar**: thin chrome above scene (account, accessibility, notebook, offer). Fades to near-transparent after ~3s cursor stillness; returns on movement/keyboard. No UI inside the scene.
- **Reduced-motion mode**: cross-fade pose sequences replace frame animation; flight → cross-fade between perches; ambient drift removed; color transitions slowed but present. Triggered by `prefers-reduced-motion` or settings opt-in. A designed surface, not a stripped fallback.
- **Transitions**: settle → slow evening shift over ~4s with 5s any-click undo; listen-in → gentle camera-agnostic emphasis (slight scale/light on focused bird) plus audio mix ramp.

## 8. Audio pipeline

- **Synthesis**: WebAudio graph per bird: oscillator/noise sources shaped by motif envelopes from the call-grammar runtime; per-bird timbre offset for signature stability. No audio files shipped, ever.
- **Chorus mixing**: per-bird gain nodes into a master bus with soft limiting. Ambient bed (very quiet wind/room tone synthesized, also procedural) underneath.
- **Listen-in mix**: focused bird gain ramps up over ~1.5s, others ramp down to ambient floor (never silence); same ramp on disengage. Feels like listening, not soloing.
- **Autoplay policy**: AudioContext resumes on first user gesture; until then, aviary plays in graceful silence with captions auto-on (also the WebAudio-unavailable fallback — no recorded-audio path exists).
- **Captions**: generated client-side from the same motif+parameter set as the synthesized call ("a soft three-note rise"), rendered as small fading text near the calling bird; naturalist voice.
- **Memory**: preallocated buffer pool; no per-call allocation; AudioContext bounded — the no-memory-growth CI test covers this.

## 9. Accessibility surfaces

- **Narration**: aria-live (polite) region fed naturalist prose generated from snapshot state, ~1 update per 30–60s at idle, priority bump (still observation-voiced) for return-greeting, offer reactions, settle. Same voice as notebook. Client generates from snapshot fields via the same prose engine family as the server notebook generator.
- **Keyboard**: Tab through top bar; Tab into scene focuses first bird; arrows move between birds; Enter = listen-in; Escape = disengage; offer flow fully keyboard-navigable; settle from top bar. Visible high-contrast focus indicators against bright and dim scenes.
- **Contrast**: WCAG AA minimum for all chrome/copy; verified in CI with automated checks on settings/error/caption surfaces.
- **Reduced-motion** and **captions**: as above; settings persist server-side, honor media query by default.
- **Ship gate**: accessibility surfaces are launch-blocking, not v1.1.

## 10. Performance budgets and observability

- Budgets (CI-enforced): initial JS <2MB gzipped; first bird <500ms (synthetic fleet, mid-tier mobile profile, 4G throttle, multiple geographies); 60fps idle sustained 30 min on reference laptop; zero memory growth over 30 min (heap snapshots in CI); tick p99 <5s alarm.
- RUM: aggregate-only — load timings, first-bird timings, frame timings, audio-context error counts, tick latencies, anonymized session-duration histograms. **No per-bird state, no per-account dimensions** — metric definitions are reviewed against the privacy boundary before shipping.
- What we deliberately don't measure: anything reconstructing a user's relationship with their birds (no per-account drift dashboards, no population interaction analysis, no leaderboard substrate).

## 11. Rollout

1. **Internal dogfood**: engine + client behind flag; calibrate drift k, mood transition rates, greeting variation, notebook sparsity against the 1-week-instrument / 3-week-user targets using simulated presence schedules.
2. **Private beta**: invite-only accounts; watch tick latency, snapshot payload size, audio-context error rates, caption quality; tune presence activity window (start 4 min).
3. **Public v1**: full surface including visits (off by default), export, deletion, accessibility suite.
4. **Birds-per-aviary ramp**: starts at 2 for everyone; age-based offers (third bird ~3 months, up toward 7 over ~a year) ship enabled from day one — the ramp is time, not feature flags.
5. **Day-one instrumentation**: synthetic perf fleet, aggregate RUM, tick latency alarms, error rates; sign-in funnel health (aggregate counts only).

## 12. Risks

- **Drift calibration**: too fast → Tamagotchi; too slow → screensaver. Mitigate with simulation harness (scripted presence schedules asserting instrument-visible drift at 1 week), beta tuning, and a kill-switch config for drift rates without redeploy. Biggest silent-failure risk in the product.
- **Presence definition laxity**: any shortcut ("tab open") corrupts drift population-wide, silently. Enforce the 3-signal conjunction in one well-tested client module with server-side sanity checks (reject implausible presence durations).
- **Sync correctness**: client must never write personality; enforce via API schema (no personality fields accepted), code review lint rule, and tick-only write path in the data layer. Additive deltas in event order; idempotent tick via watermarks.
- **Audio uncanniness**: repetitive-sounding "procedural" calls break the spell as badly as loops. Mitigate with seeded variation depth (multiple jitter dimensions), chorus phase randomness, and listening tests in dogfood; per-bird timbre stability tests so recognizability survives variation.
- **Accessibility regressions**: narration collapsing into state-list phrasing, reduced-motion becoming "animations off." Mitigate with copy review gates, screen-reader dogfooding (VoiceOver/NVDA), and snapshot tests on narration/caption prose voice.
- **Notebook voice drift**: generic entries leak system-ness. Phrase-bank coverage tests + human review of sampled entries each release.
- **Memory leaks** in long sessions (audio buffers, notebook DOM): CI heap-growth test is a hard gate.
- **Scope creep into gamification/social**: every "harmless" streak/toast/notification proposal is rejected by reference to non_goals.md; plan includes a standing design-review checklist item for it.
