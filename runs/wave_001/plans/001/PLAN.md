# Pocket Aviary — v1 Implementation Plan

This is the implementation plan for Pocket Aviary v1: a browser-only aviary where a few birds keep living on a server-side simulation, notice the user, and drift slowly over weeks in response to the user's presence. It turns the PRD into architecture, data, protocols, algorithms, budgets, tests, and a rollout sequence that a separate team can build from.

Terminology follows the PRD glossary: *bird*, *aviary*, *call*, *mood*, *personality vector*, *drift*, *presence*, *listen-in*, *offer*, *settle*, *field notebook*, *visit*, *tick*. This plan adds a few internal terms, defined where they first appear and collected in §19.

Decisions this plan makes where the PRD is silent or contradicts itself are tagged **D-nn** and collected in §1.4.

---

## 0. The hard rules and how they are enforced

The PRD's affective promises only hold if they survive a year of well-meaning contributions. We turn each one into an invariant that architecture, database permissions, lint rules, or CI tests enforce. Reviewer memory isn't enough.

| # | Invariant | Primary enforcement |
|---|---|---|
| H1 | Only the server tick writes personality and mood. Traits never decrease. Trait values never reach any product surface. | DB column grants (only the tick role can UPDATE trait columns); a trigger that rejects any trait decrease; a snapshot contract test that fails if trait values appear in client payloads (§3.4, §14) |
| H2 | Presence requires `visibilityState === 'visible'` AND window focus AND pointer or key activity within the activity window. Nothing else counts, and visitors never count. | A single presence module with an exhaustive browser-automation truth-table test; server-side interval validation and a cross-device union; visitor credentials cannot reach the events endpoint (§5.4, §12) |
| H3 | Nothing announces. No toasts, welcome text, badges, unread dots, streaks, counters, push, or re-engagement email. | Copy lint on banned phrases and registers; banned-API lint (`Notification`, `PushManager`); an anti-announcement checklist in the PR template; a static document title and favicon (§14) |
| H4 | Aliveness. Calls are always procedural and no recorded audio ships anywhere. The first frame is mid-action. There are no spinners, no canned greetings, and no identical repeats. | An asset-pipeline check that rejects audio files; a first-frame visual test; a no-repeat test on greetings and calls (§7, §8, §14) |
| H5 | Privacy. Accounts are synthetic UUIDs everywhere. Email lives encrypted in exactly one table. Per-account interaction data never leaves the simulation database for any aggregate, analytic, ML, or third-party purpose. | Separate databases and credentials; no network path from analytics to the simulation DB; logging and telemetry field allowlists; a PII lint (§11) |
| H6 | Accessibility ships in v1 as designed surfaces: naturalist narration, a reduced-motion rendering, captions generated from the call grammar, and full keyboard reach. | Launch gate; a type-level rule that every animation declares its reduced-motion variant; voice lint on narration and captions (§10, §15) |

---

## 1. Scope

### 1.1 In v1

- **Aviary.** One per account and one horizontal scene. Two starter birds chosen by the system, growing to seven by aviary age (§5.14). The species pool has six species, one of them a nightjar-like night caller.
- **Bird engine.** The five-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic drift, the enumerated mood model, a procedural call grammar per bird, mood-shaped idle motion, bird-to-bird interaction and chorus, weather, and a local-time day/night cycle.
- **Interactions.**
  - A return-greeting that varies with absence length, boldness, and mood.
  - Listen-in with a gradual re-balance of the mix.
  - Offers: a seed, a song fragment from a small library, or a still pool. Each bird has its own cooldown.
  - Settle, with a five-second undo.
  - A read-only field notebook, written sparsely.
  - Presence accounting.
- **Accounts.**
  - Email magic-link sign-in: links expire after 15 minutes and are single-use, with per-email rate limits.
  - Per-device sessions that can be revoked.
  - Email change that is verified before it is committed.
  - JSON export delivered as an emailed link.
  - Deletion that is soft for 30 days, then hard.
  - Synthetic account UUIDs throughout.
- **Sync.** A server-side tick at roughly one per minute. Clients pull snapshots. Clients append to an append-only event log. Personality has no last-write-wins path.
- **Social.** Visit invitations, one email per invite, read-only and ambient. Visits are off by default, revocable, and unused invites expire after 30 days. There is a visit log in settings and an opt-in visit-notification toggle, off by default.
- **Accessibility.**
  - Screen-reader narration in naturalist prose.
  - A designed reduced-motion mode.
  - Call captions generated from the grammar.
  - WCAG AA contrast.
  - Full keyboard navigation.
- **Performance.**
  - Initial JS under 2 MB gzipped, with a much lower internal target.
  - First bird visible in under 500 ms.
  - 60 fps idle motion on a five-year-old laptop.
  - No memory growth over 30 minutes, enforced in CI.
  - Silence with captions when WebAudio is unavailable.
- **Other.** A matter-of-fact unsupported-browser surface. Aggregate-only operational telemetry, synthetic monitoring, and RUM.

### 1.2 Refused, and enforced as refusals

These are not missing features. We refuse them, and we make each refusal hard to reverse by accident.

| Refused | How the refusal is made durable |
|---|---|
| Gamification: achievements, streaks, levels, scores, badges, XP, "birds adopted: N", visit calendars | No per-account visit-frequency data is exposed through any API. The copy lint bans the vocabulary. Notebook detectors are forbidden from observing user behavior (§9.3). |
| Tamagotchi mechanics: death, hunger, distress, happiness decay | There is no code path that deletes birds (the only DELETE grant is the account-purge role). The mood set has no distress state. Absence is not an input to wary. Traits cannot decrease (H1). The pose library has no negative-affect poses. |
| Social-network surfaces: profiles, follows, feeds, discovery, comments, chat, avatars, leaderboards, mutual visits | Visitor passes are never linked to accounts, even when the emails match. No cross-account statistics are computed. Visitor credentials are read-only by construction (§12). |
| Notifications: push, ping, re-engagement email | No Push/Notification API code (lint). The email provider is configured for transactional messages only, with no marketing lists. The only opt-in email is the visit notification, off by default. |
| Native apps | Web only. We don't design protocols around native constraints and don't build installability features beyond a basic web manifest. |
| Payments, shared aviaries, multiple aviaries per account, custom scenes, user-arranged perches | These are not in the data model: one aviary per account (enforced with UNIQUE), no perch-placement API, no billing tables. |
| Exposing trait numbers anywhere in the product | H1. The only exception is the account export (D-02). |
| Recorded audio | Asset-pipeline gate. The WebAudio fallback is silence with captions. |
| Removing or "releasing" a bird | Not in the PRD, and it would contradict identity continuity. Birds persist for the life of the account. |

### 1.3 Deferred, not refused

- SSO and password login.
- Localization. The naturalist grammar is English-specific, and v1 is English only.
- Multi-region writes. v1 has one primary region, a global edge, and regional read caches as a lever (§13.6).
- Raising the seven-bird cap. That needs future work on audio recognizability.
- A self-hosted language model for prose variety. Third-party LLMs are ruled out (D-14).

### 1.4 Interpretive decisions (resolving ambiguity or conflict in the PRD)

| ID | Tension in the PRD | Decision | Why |
|---|---|---|---|
| D-01 | The top bar holds four items, "nothing else". Separately, settle is "triggered from the top bar". | The top bar keeps exactly four icons. The **offer** icon opens a small *gesture tray*: offer a seed, a song fragment, or a still pool, then a separator and *settle*. Settle also has a keyboard shortcut. | This meets both requirements literally. Settle is a gesture toward the aviary, like an offer, so it belongs beside them rather than in the system-voice settings menu. |
| D-02 | Traits are "never exposed numerically", yet the export includes "current personality vectors". | The export contains the vectors as raw JSON numbers, because the PRD lists them explicitly and data portability likely requires it. The export is never rendered, summarized, or visualized anywhere in the product. It is a file that only arrives through an emailed link. | The export rule is specific and later; the never-exposed rule governs product surfaces. Product and legal should confirm this (§18). |
| D-03 | interactions.md says keyboard-focusing a bird listens in. accessibility_perf.md says arrows move focus and Enter triggers listen-in. | Focus alone never engages listen-in. Enter or Space engages it. Moving focus to another bird or out of the scene disengages it, and so does Escape. | If focus engaged listen-in, arrowing across birds would thrash the mix, which feels like switching channels — the thing the PRD forbids. |
| D-04 | "Calls already audible" on the first frame, versus browser autoplay policies. | Create the AudioContext at boot. If the browser allows it to run (site engagement, permissions), calls are audible from the first frame. If it is suspended, the first trusted gesture resumes it, fading in over about 1.5 s into a chorus that was already in progress. There is no "tap for sound" prompt. | We can't override browser policy. A prompt would announce; a join-in-progress keeps the illusion. Tracked as risk R-07. |
| D-05 | Neglected birds become "quieter", but traits never decrease. | Add a per-bird **attention** state: a low-passed record of recent presence, listen-in, and offers. It decays during absence and modulates expression only: how eagerly and how often a bird greets, and how far it approaches. Traits, base call rate, plumage, and mood valence are untouched. Absence is never an input to wary. | This is the literal reading of "greeting less often because less often is what's been observed". It also gives a returning user something to ease back into. |
| D-06 | The brief says birds drift with "whether you mute the calls". The engine's list of drift inputs omits mute. | Mute is **not** a drift input in v1. | Weighting drift by audio would ration the product by sensory ability, which contradicts the accessibility stance. The engine file is the authority on mechanics. |
| D-07 | "keypress" is a deprecated event, and touch devices may not emit pointermove on a tap. | Count trusted `pointermove`, `pointerdown`, and `keydown` (`event.isTrusted` only). Nothing else counts: no scroll, wheel, resize, or programmatic events. | `pointerdown` is at least as strong a sign of presence as a move, so this doesn't loosen the definition. |
| D-08 | "The visitor sees exactly what the host would see". | Visitors see the canonical aviary in the **host's** time zone, with the default ambient mix. Host session-local states are not mirrored: listen-in mix, settle lighting, greetings. Visitors get no notebook and no settings other than their own local accessibility preferences. | Mirroring session-local states would amount to co-presence. The notebook is the host's private record. |
| D-09 | Offers are not aimed at a bird, but "the receiving bird's mood" shapes the reaction. | The server picks the responding birds by curiosity, mood, proximity, attention, and cooldown (§5.12). | This fits the rule that offers can't be made by clicking a bird. |
| D-10 | Narration could be generated on the server or the client. | Generate it on the client, from the same snapshot and choreography state the renderer uses, with a shared voice package (§9). | It needs no extra round trips, keeps timing with visible motion, and uses one grammar for narration, captions, and (on the server) the notebook. |
| D-11 | "Rain dampens vocal frequency". | Rain is an expression modifier on call rate during and shortly after rain. It never changes a trait. | Traits are monotonic. |
| D-12 | The lifetime of an active (used) invitation is unspecified. | Active until the host revokes it. It lapses after 90 days without a visit. | This stops abandoned passes from turning into a permanent visitor list, which matches the rationale of the 30-day expiry for unused invites. |
| D-13 | "Settled" is both an aviary lighting state and the night state of a bird. | The internal bird mood enum is `roosting`. Prose may say "settled for the night". The aviary's settle state is `session.settled`. | This avoids overloading one identifier in code. |
| D-14 | Prose could be generated by an LLM. | No third-party model receives any aviary state. Notebook, narration, and captions use in-house grammars. | "Never shared with any third party." |
| D-15 | Hidden tabs versus unfocused-but-visible windows. | When hidden, rendering stops and audio fades out and suspends. When visible but unfocused (a second monitor, say), rendering and audio continue but **no presence accrues**. | This is the literal presence rule. The user can still glance at it; the engine still doesn't count it. |
| D-16 | "Return from another tab" greets, but rapid tab-switching would turn greetings into a reflex. | Greet only after the tab has been hidden for at least 20 s (configurable), after a new navigation, or after a suspend gap. | A greeting that fires on every alt-tab reads as canned. |
| D-17 | Time of day for the canonical mood across devices in different zones. | The account has a canonical IANA time zone, updated from whichever device most recently accrued presence. All rendering, including for visitors, uses the aviary's time zone. | Keeps "same aviary, same mood" true across devices. |
| D-18 | The invitation email has to identify the host, and there are no profiles. | The email names the host by their account email address, which the host consents to when sending. | There are no display names or profiles to use. |
| D-19 | Settling at night would "shift to evening", which is a brightening. | Settle lighting targets the dusk palette only when the scene is currently brighter than dusk. Otherwise it deepens and warms the current palette slightly. | Settle should never wake up a night scene. |
| D-20 | The top bar fades on cursor stillness, but touch devices have no cursor. | On touch devices, the bar fades after 4 s without touches. A tap in its region while it's faded reveals it without activating anything. | Prevents accidental activation of controls the user can't see. |

---

## 2. Architecture

### 2.1 System overview

```
┌────────────────────────── Browser (web client) ─────────────────────────────┐
│ Inline boot (quiet-field painter, feature detect, unsupported surface)      │
│ Scene core: choreographer (pure fn of timeline × time) → Canvas2D renderer  │
│ Audio engine: AudioWorklet "syrinx" synth + per-bird channel strips         │
│ Voice runtime: narration composer, caption composer (shared voice package)  │
│ Presence tracker → Event outbox (memory + IndexedDB, beacon on pagehide)    │
│ Panels (code-split): notebook, account/settings, accessibility, visits      │
│ Service Worker: shell + asset cache, last snapshot, navigation preload      │
└───────────────▲──────────────────────────────────────────────┬──────────────┘
                │ HTTPS (HTTP/3), cookie session                │ aggregate RUM beacons (no ids)
┌───────────────┴────────── Edge (CDN + edge function) ─────────▼──────────────┐
│ Streams HTML shell immediately (103 Early Hints for scene core),            │
│ fetches the snapshot from origin in parallel and streams it inline;         │
│ serves immutable hashed assets; RUM ingest strips IP / identifiers.         │
└───────────────▲──────────────────────────────────────────────────────────────┘
                │
┌───────────────┴──────────────── Regional origin ────────────────────────────┐
│ API service (stateless TS): auth, snapshot read + projection, event ingest, │
│   offer resolution, greeting selection, notebook read, settings, visits     │
│ Tick scheduler + tick workers: THE ONLY WRITER of canonical sim state       │
│ Notebook worker: happenings → sparse entries                                │
│ Jobs: export, deletion purge, invite expiry/lapse, email change, cleanup    │
│ Postgres "auth_db": accounts (encrypted email), links, sessions, invites    │
│ Postgres "sim_db": aviaries, birds, events, notebook, happenings            │
│ Redis/Valkey: snapshot cache, rate limits, session cache                    │
│ Object storage: export files (encrypted, 7-day lifecycle)                   │
│ KMS: per-account data keys, email blind-index pepper                        │
│ Transactional email provider (primary + secondary)                          │
└─────────────────────────────────────────────────────────────────────────────┘
┌──────────── Ops telemetry plane (separate account/VPC) ─────────────────────┐
│ Metrics, logs (allowlisted fields), traces, RUM aggregates, synthetic       │
│ monitoring. NO credentials for and NO network route to sim_db.             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Client/server split

| Concern | Server (authoritative) | Client (presentation only) |
|---|---|---|
| Personality vector, attention, cooldowns | Stored, updated by the tick, never sent to the client | Never sees them |
| Mood | Tick transitions; snapshot carries the enum | Shapes idle style and narration phrasing; never labeled on screen |
| Perch and activity plan | Tick writes an *intent timeline*: coarse, timestamped intents for the next ~5 min | The *choreographer* expands intents into continuous motion |
| Call timing | Tick schedules ambient calls with seeds and chorus coupling | The grammar expands (signature, mood, seed) into a CallScore, synthesizes it, and captions it |
| Greeting, offer reactions | API computes them at request time with engine functions, logs an event, returns a choreography descriptor | Plays the descriptor |
| Weather, time zone, daylight | Tick | Renders them |
| Leaves, feathers, parallax sway, micro-motion, flight paths, cross-fades | — | Purely client-side ornament with no per-leaf state (per PRD) |
| Listen-in mix, settle lighting, top-bar fade | Receives events only (drift and mood inputs) | Session-local presentation |
| Presence | Validates, unions across devices, credits | Detects the three-signal conjunction and sends heartbeats |

### 2.3 Where the render pipeline ends

The server authors **what happens and when**: mood, perch changes, activity modes, call events with seeds, scene objects, and weather. The client authors **how it looks and sounds**: rig poses, flight arcs, noise-driven micro-motion, synthesis, captions, narration text, and ornaments.

The choreographer is a pure function `render_state = f(timeline, seeds, t)`. The client never mutates canonical state and never "ticks". The protocol types for snapshot data are deep-readonly, so it can't. Because rendering is a function of time, an open tab can show a bird mid-preen at the right phase on the first frame. For the same reason, two devices or a visitor see the same perch changes and calls at the same moments, after clock offset.

### 2.4 Technology choices

- **TypeScript end to end.** The engine, call grammar, voice grammar, and protocol are shared packages used by the client, the API, the workers, and test tooling. Services run on the current Node LTS in containers.
- **Seeded PRNG.** All randomness in shared code comes from a seeded, integer-based PRNG (sfc32/xoshiro128**), so replays and cross-environment tests are deterministic. Server-side decisions never depend on bit-exact transcendental results matching across JS engines.
- **Client.** No framework on the scene path: a custom Canvas2D renderer and choreographer. Preact (about 4 KB) for panels, code-split. Built with Vite/Rollup, with `size-limit` budgets in CI.
- **Datastores.** Postgres 17+ with high availability: a synchronous standby, WAL archiving, and PITR. Redis/Valkey for caches and rate limits.
- **Edge.** An edge function platform that supports streaming responses and 103 Early Hints, such as Cloudflare Workers or Fastly Compute.
- **Testing.** Vitest, fast-check (property tests), Playwright with CDP access (presence, performance, memory, visual tests), axe-core, and a custom audio lab built on OfflineAudioContext.

### 2.5 Repository layout (monorepo)

```
packages/
  protocol/      # API + snapshot + event schemas (zod), deep-readonly types, version constants
  engine/        # PURE: tick(), drift, attention, mood, weather, planner, greeting, offer resolution,
                 #       genesis/adoption, happenings; seeded RNG; no I/O
  callgrammar/   # PURE: species motif libraries, bird signatures, CallScore + CaptionDescriptor
  voice/         # PURE: phrase banks, prose grammar (narration, captions, notebook), voice lint
  choreo/        # PURE: timeline × t → poses/positions; reduced-motion pose sequencer
apps/
  web/           # client: boot, scene, audio, presence, panels, SW
  edge/          # HTML streaming + early hints + RUM scrub
services/
  api/  tick-worker/  notebook-worker/  jobs/
tools/
  sim-harness/   # calibration personas, long-horizon simulation, reports
  audio-lab/     # offline render, recognizability classifier, listening-test builder
  voice-corpus/  # mass-generates prose for lint + human review
infra/           # IaC, DB migrations (with grants/triggers), dashboards, alerts
```

---

## 3. Data model

### 3.1 auth_db: identity, email, sessions, invitations

This is the only database that holds email addresses. Every email column is encrypted with the account's data key (envelope encryption via KMS).

| Table | Key fields | Notes |
|---|---|---|
| `accounts` | `account_id uuid PK` (random v4), `email_ct bytea`, `email_bidx bytea UNIQUE`, `status` (`active`, `pending_deletion`), `deletion_requested_at`, `purge_after`, `created_at`, `dek_wrapped`, `settings jsonb` | `email_bidx` = HMAC-SHA256(KMS pepper, normalized email). Normalization lowercases and trims; it does not strip dots or plus tags. The blind index exists only for sign-in lookup and rate limiting and never leaves this service. Settings hold accessibility preferences, sound, captions, narration, shortcuts, and the visit-notification toggle (default off). |
| `magic_links` | `token_hash PK` (SHA-256 of a 256-bit token), `purpose` (`sign_in`, `email_change`), `account_id NULL`, `pending_email_ct` (first sign-up), `expires_at` (+15 min), `consumed_at`, `created_at` | Consumed atomically: `UPDATE … SET consumed_at=now() WHERE token_hash=$1 AND consumed_at IS NULL AND expires_at>now() RETURNING`. Rows are purged 24 h after expiry. |
| `sessions` | `session_id uuid`, `account_id`, `token_hash`, `device_label` (e.g. "Firefox on macOS", from the UA family), `created_at`, `last_seen_date` (day granularity), `expires_at` (90-day sliding, 1-year absolute), `revoked_at` | No IP addresses or geolocation are stored. |
| `email_changes` | `id`, `account_id`, `new_email_ct`, `new_email_bidx`, `token_hash`, `expires_at`, `verified_at` | The old email keeps working until the new one verifies. |
| `visit_invitations` | `invitation_id uuid`, `host_account_id`, `visitor_email_ct` (host's key), `link_token_hash`, `pass_token_hash NULL`, `status` (`outstanding`, `active`, `revoked`, `expired`, `lapsed`), `created_at`, `expires_at` (+30 d while unused), `claimed_at`, `last_visit_at`, `revoked_at` | Never joined to `accounts` by email (§12). |
| `visit_sessions` | `id`, `invitation_id`, `started_at`, `last_heartbeat_at` | Exists only for the host's visit log (approximate duration). Kept for 12 months. |

### 3.2 sim_db: the canonical aviary

| Table | Key fields | Notes |
|---|---|---|
| `aviaries` | `aviary_id uuid PK`, `account_id uuid UNIQUE`, `created_at` (drives bird availability), `tz`, `tz_updated_at`, `engine_version`, `tick_seq`, `last_tick_at`, `event_cursor bigint`, `rng_seed`, `weather jsonb`, `scene_objects jsonb`, `timeline jsonb`, `committed_until`, `next_newcomer_at`, `max_birds_override NULL`, `version` | A single row per account. The timeline holds only future intents. |
| `aviary_event_heads` | `aviary_id PK`, `next_seq bigint` | Serializes event sequence numbers per aviary (§6.3). |
| `birds` | `bird_id uuid PK`, `aviary_id`, `species_id`, `species_rev`, `name`, `adopted_at`, `arrived_at`, `voice_signature jsonb` (immutable), `plumage_base jsonb` (immutable), `t_boldness`, `t_warmth`, `t_vocal`, `t_plumage`, `t_curiosity` (real, 0..1), `attention jsonb` (per-channel filter states), `daily_credit jsonb` (per-day saturation accumulators), `mood` (`alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`), `mood_since`, `mood_ctx jsonb` (decaying modifiers), `perch_zone`, `perch_slot`, `offer_cooldown_until`, `version` | `bird_id`, `species_id`, `voice_signature`, and `plumage_base` are immutable (no UPDATE grant). |
| `bird_trait_daily` | `(bird_id, day)`, five traits | An integrity snapshot for targeted restore (§6.7). Daily rows for 180 days, then weekly. It is **never** used to derive runtime state. |
| `events` | `(aviary_id, seq)` PK, `event_id uuid` (UNIQUE per aviary), `type`, `bird_id NULL`, `payload jsonb`, `outcome jsonb`, `device_id`, `client_ts`, `received_at` | Append-only and partitioned by day. Raw events are kept 35 days and then dropped; derived state is what persists. |
| `presence_intervals` | `aviary_id`, `start`, `end` | The tick's merged union across devices. Kept 35 days, for absence length and daily credit. |
| `greetings` | `aviary_id`, `at`, `greeter_ids[]`, `form`, `fingerprint` | Anti-repeat history and notebook detectors. Kept 60 days. |
| `happenings` | `aviary_id`, `at`, `kind`, `bird_ids[]`, `attrs jsonb`, `salience` | Aviary-side observations emitted by the tick. Kept 60 days. |
| `notebook_entries` | `entry_id`, `aviary_id`, `observed_at`, `local_date`, `text`, `kind`, `bird_ids[]`, `generator_rev` | Immutable. Kept for the life of the account. |
| `phrase_usage` | `aviary_id`, `phrase_key`, `last_used_at` | Stops notebook and narration phrasing from recurring over months. |
| `newcomers` | `newcomer_id`, `aviary_id`, `species_id`, `voice_signature`, `offered_at`, `status` (`visiting`, `welcomed`, `let_go`), `resolved_at` | Newcomers are not birds until welcomed (§5.14). |

`device_id` is a random per-browser ID kept in localStorage. It is used only to de-duplicate and validate presence heartbeats and to label sessions, and it isn't linked to anything else.

### 3.3 Storage-level enforcement

- **Roles.**
  - `api_role` can INSERT into `events` and read the sim tables. Its only UPDATE is `birds.name`, guarded by a version check.
  - `tick_role` can UPDATE `aviaries` and the mutable columns of `birds`, INSERT into `birds` (for genesis and adoption), and write happenings, greetings, and presence intervals.
  - `notebook_role` can INSERT into `notebook_entries`.
  - `purge_role` is the only role with DELETE, and it is used only by the hard-delete job.
- **Monotonicity trigger.** `BEFORE UPDATE ON birds` raises an error if any `NEW.t_* < OLD.t_*`, unless the session variable `aviary.repair_mode` is set. That variable is available only to an audited break-glass repair role (§6.7).
- **Isolation from analytics.** Analytics and observability services get no role on sim_db, and network policy blocks the route entirely.

### 3.4 What leaves the server

Snapshots carry **derived, quantized expression parameters**, never trait values:

- Plumage arrives as computed colors and a detail level.
- Idle style arrives as mood-and-personality-mixed weights, quantized to 8 levels.
- Boldness and vocal frequency never appear. They only shape the server's perch and call timeline.

A contract test serializes thousands of snapshots from simulated aviaries and fails if any numeric field equals a stored trait value to four decimal places, or if any field name matches a trait name. This is defense in depth for H1: devtools isn't a product surface, but a plain `boldness` key in client code is how "show my bird's stats" features start.

### 3.5 Retention summary

| Data | Retention |
|---|---|
| Birds, traits, names, signatures, notebook | Life of account |
| Raw interaction events, presence intervals | 35 days |
| Greetings, happenings, phrase usage | 60 days |
| Trait daily snapshots | 180 days daily, then weekly, for life of account |
| Visit log / visit sessions | 12 months |
| Magic links | Expiry + 24 h |
| Sessions | Revoked or expired + 30 days |
| Server logs | 14 days (UUIDs only, no payloads); traces 7 days |
| DB backups / WAL | 30 days, encrypted. A deletion ledger is replayed on any restore (§11.6). |
| Exports | 7 days in object storage |
| Accounts pending deletion | 30 days, then everything above is purged |

---

## 4. API surface

### 4.1 Conventions

- JSON over HTTPS (HTTP/3 preferred), versioned under `/v1`.
- Sessions use an HttpOnly, Secure, SameSite=Lax cookie holding an opaque session token.
- Mutating calls require `Origin` to match and a custom `X-Aviary-Client` header, as CSRF defense in depth.
- Every response carries `server_now` (epoch ms) for clock-offset estimation.
- Event and offer submissions carry client-generated UUIDv7 `id`s for idempotency.
- Schemas live in `packages/protocol`. The client supports snapshot schema N and N−1. On a major mismatch it reloads at the next visibility change, never mid-view and never behind a spinner.
- All user-facing error strings are system-voice copy keys (§9.5).

### 4.2 Endpoints

**Auth and account.** System voice; code-split UI.

| Method | Path | Purpose |
|---|---|---|
| POST | `/v1/auth/link` `{email}` | Always returns 202 with the same body whether or not the account exists. Rate-limited per blind index and per IP prefix. |
| GET | `/auth/verify?t=…` | A landing page that does **not** consume the token. It auto-submits a POST via JS, with a fallback button. This defeats corporate link scanners that pre-fetch GET URLs. |
| POST | `/v1/auth/verify` `{token}` | Atomically consumes the link, creates the account and aviary genesis on first sign-in, sets the session cookie, and redirects to `/` (or the naming step for new accounts). |
| POST | `/v1/auth/signout` | Revokes the current session. |
| GET / DELETE | `/v1/account/sessions`, `/v1/account/sessions/{id}` | Lists sessions (device label, last-active date, "this device"). Revocation takes effect immediately (§11.2). |
| POST | `/v1/account/email-change` `{new_email}` → `/v1/account/email-change/verify` `{token}` | Verify first, then commit. |
| POST | `/v1/account/export` | Returns 202. A job builds the JSON and emails a download link. |
| GET | `/v1/account/exports/{id}` | Requires a session for the same account plus a signed export token. Expires after 7 days. |
| POST | `/v1/account/delete`, `/v1/account/restore` | Soft delete; "I changed my mind" restore. |
| GET / PATCH | `/v1/account/settings` | Per-field merge patch. |

**Aviary** (host only).

| Method | Path | Purpose |
|---|---|---|
| GET | `/v1/aviary/snapshot?reason=open\|visible\|resume\|keepalive` | Canonical state plus projection plus timeline. `ETag` is `tick_seq.projection_seq`, and keepalives may get 304. For `open`, `visible`, and `resume` it can include a `greeting` (§5.11). |
| POST | `/v1/aviary/events` `{device_id, events:[…]}` | Batch append of `presence`, `listen_in`, `settle`, `greeting_seen`, and `tz_report`. Per-event acks. Duplicate IDs return the original ack. |
| POST | `/v1/aviary/offers` `{id, kind, fragment_id?}` | Resolves reactions synchronously (§5.12) and returns choreography descriptors. |
| PUT | `/v1/birds/{bird_id}/name` `If-Match: version` | Rename. A 409 on a stale version triggers a system-voice notice (§6.6). |
| POST | `/v1/aviary/starters` `{names:[…]}` | Names the two starter birds after genesis. Defaults are pre-filled. |
| POST | `/v1/aviary/newcomer/{id}` `{action: welcome\|let_go, name?}` | Writes an event. The tick performs the adoption (§5.14). |
| GET | `/v1/notebook?before=cursor&limit=20` | Newest first, cursor-paginated, unlimited history. |

**Visits.**

| Method | Path | Purpose |
|---|---|---|
| GET | `/v1/visits` | Host: the visit log (visitor email, date, approximate duration, newest first) plus outstanding invites. |
| POST | `/v1/visits/invitations` `{email}` | Host: invite one named visitor. Rate-limited (10/day). |
| DELETE | `/v1/visits/invitations/{id}` | Host: revoke. Takes effect immediately. |
| GET | `/visit/{public_id}?t=…` | Visitor landing page (no consumption on GET), then `POST /v1/visit/claim`. |
| GET | `/v1/visit/snapshot` | Visitor snapshot with visitor-pass auth, or 410 with the "no longer available" copy. |
| POST | `/v1/visit/heartbeat` | Duration logging only. Writes `visit_sessions` in auth_db and never touches sim_db. |

### 4.3 Snapshot shape (illustrative)

```json
{
  "schema": 1,
  "server_now": 1790000000000,
  "tick_seq": 812345, "projection_seq": 3,
  "aviary": { "tz": "America/Chicago", "daylight": { "sunrise_min": 402, "sunset_min": 1131 },
              "weather": { "kind": "rain", "intensity": 2, "from": 1789999700000, "until": 1790000500000 },
              "scene_objects": [ { "kind": "still_pool", "slot": "front", "until": 1790000420000 } ] },
  "config": { "activity_window_ms": 240000, "heartbeat_ms": 30000, "keepalive_ms": 60000,
              "greet_min_hidden_ms": 20000 },
  "birds": [ {
      "id": "b_7f…", "name": "pip", "species": "warbler", "species_rev": 1,
      "plumage": { "base": "#8b9aa2", "accent": "#c7b07a", "detail": 3 },
      "voice": { "sig": "v1:…compact signature…" },
      "mood": "content",
      "style": { "preen": 5, "scan": 2, "tilt": 4, "fluff": 1, "tempo": 3 },
      "arrived": true } ],
  "timeline": [
      { "t": 1790000003000, "b": "b_7f…", "k": "fly", "to": ["front", 1], "dur": 1900, "seed": 88121 },
      { "t": 1790000011400, "b": "b_7f…", "k": "call", "type": "ambient", "seed": 29384 },
      { "t": 1790000012000, "b": "b_2c…", "k": "act", "mode": "scan", "dur": 22000, "seed": 5510 } ],
  "committed_until": 1790000045000, "horizon_until": 1790000300000,
  "greeting": null,
  "newcomer": null
}
```

- Expected size is about 2–3 KB gzipped for seven birds with a five-minute horizon, with a hard budget of 8 KB.
- The visitor variant drops `greeting` and `newcomer` and adds `audience: "visitor"`.

### 4.4 Event payloads

- `presence {id, start, end}`: a contiguous interval in server time during which all three signals held. Emitted at most every 30 s, and immediately when presence is lost.
- `listen_in {id, bird, start, end}`: emitted on disengage, and as partial intervals every 30 s during a long listen-in so a crash loses at most 30 s.
- `settle {id, at}`: posted when the five-second undo window closes, or by beacon on `pagehide` (§5.13).
- `greeting_seen {id, greeting_id}`: confirms the greeting actually played, for notebook accuracy. A greeting that was returned but never played (tab hidden again) is not observed.
- `tz_report {id, tz}`: sent at session start and whenever `Intl` reports a change.

---

## 5. Simulation engine

### 5.1 The engine is a pure package

`packages/engine` exports pure functions:

- `tick(state, events, now, rng) → {state', timelineAppend, happenings, metrics}`
- `selectGreeting(state, absence, localTime, rng)`
- `resolveOffer(state, pendingOutcomes, offer, rng)`
- `genesis(seed)`, `adoptNewcomer(state, newcomer, name)`

It has no I/O, reads no clock (`now` is always passed in), and draws all randomness from a seeded RNG. The same code runs in the tick worker, the API (greeting and offer resolution), the calibration harness, and property tests. `engine_version` is stored per aviary. Constant changes ship as new engine versions, rolled out by cohort (§15.5).

### 5.2 Tick scheduling

- **Cadence.** Nominally 60 s per aviary. Each aviary's phase offset is `hash(aviary_id) mod 60 s`, so load is smooth rather than spiking on the minute. The tick runs for every aviary whether or not a client is connected, including accounts pending deletion, so a restored aviary has been continuing too.
- **Ownership.** Aviaries map to 1,024 logical shards. Workers hold shard leases (Postgres advisory locks with heartbeats). A due-queue per shard is ordered by `next_tick_at`.
- **Transaction per aviary.**
  1. `SELECT … FOR UPDATE` on the aviary and its birds.
  2. Read events with `seq > event_cursor`.
  3. Run `tick()`.
  4. Write the state, advance `event_cursor`, bump `tick_seq` and `version`, and append happenings and presence intervals.
  5. Write an outbox row.

  After commit, an outbox relay writes the snapshot cache with compare-and-set on `tick_seq`, so an older tick can never overwrite a newer one.
- **dt-correctness.** `tick()` integrates over an arbitrary `dt` using fixed internal substeps: 60 s for mood, weather, and planning, and closed form for filter decay and drift where possible. Irregular ticks therefore give the same results as regular ones. This makes three things safe:
  - expedited ticks, scheduled about 5 s after a user interaction so mood effects of an accepted offer land promptly;
  - catch-up after an outage, which runs up to 6 h at 1-min substeps and uses 5-min substeps beyond that, prioritizing aviaries with recent snapshot reads;
  - a future tiered cadence for dormant aviaries, which is a cost lever only and produces identical results.
- **Budgets.** Target p99 tick duration is ≤ 250 ms. p99 above 5 s alerts (PRD). Tick lag (`now − last_tick_at`) has an SLO of p99 < 90 s.
- **Capacity.** 100k aviaries ≈ 1.7k ticks/s. That is manageable with batched reads and writes: `UNNEST` updates, 50–200 aviaries per worker batch with a transaction each. 1M aviaries needs horizontal worker scaling and sim_db partitioning by aviary hash. We plan that at the 250k-aviary mark (§15.4).

### 5.3 Tick pipeline (order matters)

1. **Ingest events** since the cursor, in `seq` order:
   - Merge `presence` into the union interval set.
   - Merge `listen_in` into per-bird unions.
   - Apply resolved offer outcomes (already decided at request time; §5.12).
   - Record settles and seen greetings.
   - Apply `tz_report` (D-17), with hysteresis: the new zone must come from a device that has been present for ≥ 5 min.
   - Apply `welcome` and `let_go` for newcomers.
2. **Presence credit** per local day, with saturation (§5.5).
3. **Attention filters.** Low-pass update per bird per channel (§5.5).
4. **Drift.** Monotonic integration with headroom (§5.5), followed by assertions and the anomaly clamp.
5. **Weather** process advance (§5.8).
6. **Mood** transitions per 60 s substep (§5.7).
7. **Cooldowns and scene objects.** Expire offer cooldowns. Expire scene objects (a still pool lasts about 10 min, and seeds are consumed).
8. **Planner.** Extend each bird's intent timeline to `now + 5 min`: perches, activities, ambient calls, chorus, newcomer visits. Anything before `committed_until = now + 45 s` stays frozen (§6.4).
9. **Happenings.** Emit notable aviary-side observations for the notebook worker (§9.3).
10. **Newcomer schedule check** (§5.14).
11. **Commit** plus outbox.

### 5.4 Presence: client detection, server validation, cross-device union

**Client (single module, `presence.ts`).**

- `present(t)` ⇔ `document.visibilityState === 'visible'` ∧ `document.hasFocus()` ∧ `t − lastActivity ≤ W`.
- `lastActivity` is updated only by trusted `pointermove`, `pointerdown`, and `keydown` (D-07), throttled to one update per 250 ms.
- Listeners: `visibilitychange`, window `focus` and `blur`, `pagehide`, `pageshow`, `freeze`, and `resume`.
- The activity window `W` comes from the snapshot config. It starts at 4 min and is calibrated within 2–6 min, leaning longer as the PRD asks, because sitting still and watching is the product. Recalibrating needs no client deploy.
- While present, the client accumulates an interval. It emits a `presence` event every 30 s and immediately on loss: blur, hidden, activity timeout, settle commit, or `pagehide` via `sendBeacon`.
- In the settled state, presence emission stops until the user re-engages (§5.13).
- There is no presence of any kind in visitor mode. The module isn't instantiated there.

**Server (API validation plus tick).**

- Reject or trim anything that looks wrong:
  - intervals longer than 45 s;
  - `end` in the future beyond clock slack;
  - `start` more than 10 min before `received_at` (unverifiable late data);
  - overlaps from the same `device_id`.

  An aggregate counter tracks rejections.
- The tick credits the **union** of intervals across all devices. A laptop and a phone both open and present for the same ten minutes credit ten minutes, not twenty.
- Absence length for greetings is `now − max(end)` over the union, at account level, so returning to the laptop minutes after using the phone is a short absence.

**Truth-table test (CI, Playwright + CDP).** Every combination of visible/hidden, focused/blurred, and active/idle, including "visible but unfocused on a second monitor", "minimized", "laptop left open overnight", and "background tab for two days", asserts the exact credited presence. The overnight and two-day cases must credit zero.

### 5.5 Attention filter and drift function

**Inputs per local day, saturating.** Diminishing returns keep marathon sessions from accelerating drift linearly:

- **Presence**, aviary-wide, credited to every bird: `C_p(P) = 1 − e^(−P/τ_p)`, where `P` is presence minutes so far in the aviary's local day and `τ_p = 12 min`. The instantaneous credit rate during presence is the derivative, so one day's total is always below 1.
- **Listen-in**, per bird: `C_l(L) = 1 − e^(−L/τ_l)` with `τ_l = 8 min`.
- **Offer accepted**, per bird: an impulse of 0.35, drift-credited at most 3 times per bird per day.
- **Offer near**, per bird: an impulse of 0.20 for each bird in the front or middle zone at offer time, or that approaches. Also at most 3 per day.

**Low-pass filter (the PRD's "low-pass filter over presence-and-interaction signals").** Each bird keeps an attention state `a_c` per channel c:

```
τ_a · da_c/dt = −a_c + u_c(t)        τ_a = 4 days        (u_c in credits/day)
```

Updates are exact exponential decay over irregular `dt` plus impulse integration. Two properties follow. A single session barely moves the filter. And the filter's tail keeps drift going gently for a few days after the user leaves, "based on inputs from before they left" (accounts_sync), until it drains.

**Drift (monotonic, with headroom).** For each trait k:

```
dx_k/dt = η · (Σ_c W[k,c] · a_c) · (1 − x_k)^γ          η = 0.026/day (initial), γ = 2
```

| W[k,c] | presence | listen-in | offer accepted | offer near |
|---|---|---|---|---|
| boldness | 1.0 | 0.2 | 0.2 | 1.0 |
| social warmth | 0.8 | 1.0 | 0.2 | 0.2 |
| vocal frequency | 0.6 | 1.0 | 0 | 0 |
| plumage saturation | 1.0 | 0.3 | 0 | 0 |
| curiosity | 0.6 | 0.2 | 1.0 | 0.3 |

- Every term is non-negative, so `dx/dt ≥ 0` by construction. An `assert(Δ ≥ 0)` sits in the engine, and the DB trigger backs it up.
- `(1 − x)^γ` gives diminishing increments. Traits approach 1 and never reach it, which leaves room for years of drift.
- **Anomaly clamp:** Δx per bird per trait per local day is ≤ 0.02. It should essentially never fire at the calibrated constants. When it does, an aggregate counter increments with no per-account dimension.
- **Seeds:**
  - New birds start with traits in [0.15, 0.45]: a species baseline plus individual noise.
  - The two starters differ in boldness by at least 0.10, so the greeting order is legible from the start.
- **Neglect:** no input means `a_c` decays and `dx/dt → 0`. Traits hold. Nothing in the engine subtracts from a trait.

**Expression attention (D-05).** This is the only thing that decays with absence, and it is **not** a trait:

```
A_b = 1 − exp(−(a_presence + 0.5·a_listen,b) / 0.3)
```

- A regular visitor sits around A ≈ 0.7–0.8.
- After two weeks away, A ≈ 0.05. The birds are ambient: still calling at their trait-driven rates, still in ordinary moods, but greeting with less eagerness, with fewer second greeters, and coming to the front perch less often.
- Over several regular days A recovers, so the user eases back in.

**Mapping traits to behavior (what "visible" means).** Traits act only through behavior:

- boldness → the front-zone share of perch time and approach distance;
- warmth → the probability of greeting first, call-back probability, and preference for perching next to other birds;
- vocal frequency → the unobserved call hazard and readiness to join a chorus;
- plumage → color saturation and feather-pattern contrast;
- curiosity → offer approach probability and head-tilt toward sounds.

Each mapping is designed so that a trait change of **δ_vis = 0.08** produces a behavior change at roughly one just-noticeable difference. For example, about 15% more front-perch time, or saturation +6% in the design system's perceptual color space. The internal diary study validates δ_vis (§15.2).

### 5.6 Calibration harness (drift acceptance tests)

`tools/sim-harness` runs the real `tick()` forward over simulated months, at 1-min substeps, for synthetic personas. It is a required CI job for any change to engine constants, and each change produces a report.

| Persona | Behavior | Acceptance criterion |
|---|---|---|
| Regular | 5 days/week, ~10 min presence/day, one ~2-min listen-in per session, an offer every other day | Measurable by day 7: max trait Δ ∈ [0.015, 0.035]. Visible around day 21: max Δ reaches δ_vis between day 16 and day 28. |
| Heavy | Daily, 60 min/day, frequent listen-in, offers every session | No trait crosses δ_vis before day 9. The anomaly clamp never fires. |
| Single long session | One 3-hour session, then gone | Total eventual Δ per trait ≤ 0.2·δ_vis. No session ever moves a trait visibly. |
| Light | 2×/week, 5 min | δ_vis reached after day 40 (slow, but the user still matters). |
| Tab left open | Tab open and visible for 48 h, no focus or activity | Zero presence credit, zero drift attributable to it. |
| Unfocused second monitor | Visible, not focused, for 8 h/day | Zero presence credit. |
| Absent after regular | Regular for 4 weeks, then gone 2 weeks | No trait decreases. The filter tail adds ≤ 0.01. A falls below 0.1. No wary bias. |
| Multi-device overlap | Laptop and phone present simultaneously | Credit equals the single-device credit (union). |
| Offer button-masher | 50 offers/day | Curiosity drift ≤ the 3-credited-offers/day ceiling. No saturation within a session. |
| One year regular | — | Traits between 0.65 and 0.85. None pinned at 1. |

With the constants above, a first-pass analytic estimate gives the regular persona about 0.02 at day 7, about 0.08–0.09 at day 21, and about 0.79 at one year. The heavy persona reaches δ_vis around day 10. The harness is the authority and the constants will be tuned against it. **Calibration errs slow at launch.** Drift that is too slow can be fixed going forward. Drift that is too fast can't be undone without decreasing traits, and that violates the product (R-01).

### 5.7 Mood model

- **States:** `alert`, `curious`, `content`, `wary`, `drowsy`, and `roosting` (night; D-13). There is deliberately no distress, hunger, sadness, or anger state.
- **Per 60 s substep, for each bird:** compute target scores `s_m` for every mood m:
  - **Time-of-day prior** in the aviary's local time: alert in early morning, curious and content around midday, drowsy toward dusk, roosting at night. The nightjar-like species has an inverted prior: active and calling into late evening and night, drowsy through midday.
  - **Personality terms.** Boldness lowers wary (and speeds recovery from it). Curiosity raises curious. Warmth raises content when neighbors are close.
  - **Recent-event modifiers**, stored in `mood_ctx` as exponentially decaying terms:
    - offer accepted → content (half-life 20 min);
    - a listen-in → mildly content or curious for that bird;
    - a nearby alarm call → wary (half-life 4 min, scaled by 1 − boldness);
    - rain → drowsy or content, plus a call-rate dampener (D-11);
    - a wind gust → alert for bold birds, wary for others;
    - settle → drowsy or content, calls quiet (half-life 45 min).
  - **Inertia:** +0.6 for the current mood.
- **Transitions:** switch to the target with hazard `λ = 1/(18 min)` when the target differs and the minimum dwell time has passed. Minimum dwell is 10 min. Alarm-driven entries into wary may preempt the dwell time.
- **Resulting pace.** Moods change over tens of minutes, not seconds, so the user reads mood from motion rather than watching it flicker.
- **"Daily-ish reset".** Night roosting and dawn waking (staggered 20–40 min across birds) bring each bird back to a morning distribution driven by personality. This happens continuously on the server, so the mood on tab open is always where the simulation left it and never snaps.
- **Anti-Tamagotchi rule.** Absence length and presence deficits are not inputs to any mood score. A test asserts that the mood distribution for birds with no presence for 30 days matches the distribution for birds with daily presence, apart from attention-driven greeting behavior.
- **Wary spreads**, per the PRD, via the alarm-call modifier on nearby birds, attenuated by distance and boldness. Wary is expressed as sitting further back and scanning more. It is never cowering or distressed.

### 5.8 Weather and day/night (server side)

- **Daylight.** Sunrise and sunset come from the aviary's IANA zone plus the representative coordinates in tzdb's `zone1970.tab` and the date. That gives seasonally plausible light, including the southern hemisphere, without asking for or storing the user's location. Fixed 06:30 and 19:00 is the fallback for zones without coordinates. DST comes from the tz database.
- **Rain.** A Poisson process at about 3 per week, weighted toward plausible hours. Each shower lasts 5–20 min at a soft intensity from 1 to 3. There are no thunderstorms, no snow, and nothing assertive.
- **Wind.** Occasional soft gusts, about 1–2 per day, lasting 1–4 min.
- **Mood effects** are short-lived through `mood_ctx`: during the weather and up to about 15 min after.
- Weather state is in the snapshot, and the client renders it.

### 5.9 Behavior planner (intent timeline)

For each bird, the planner appends to the timeline up to the five-minute horizon.

- **Perch choice.** The scene has three zones: front with 3 slots, middle with 4, and back with 4. That fits seven birds plus a visiting newcomer.
  - Front-zone utility rises with boldness, attention A, and curious or alert mood, plus the presence of an offered object.
  - Back-zone utility rises with wary or drowsy mood and low A.
  - Warm birds prefer slots next to occupied ones.
  - Moves happen at a mood-dependent hazard (alert and curious birds move more). A move samples a target slot by softmax, resolves slot conflicts deterministically, and emits a `fly` intent with a duration of 1.2–2.5 s scaled to distance.
  - At night, birds go to roost slots. The nightjar may move and call.
- **Activity modes:** preen, scan, rest, tilt-and-watch, forage-look, shuffle, and fluffed-rest. These are chosen from mood-weighted distributions and last 8–60 s. Each mode carries a seed that the client's choreographer uses to generate the micro-motion.
- **Ambient calls.** Each bird has a call hazard with self- and mutual excitation (Hawkes-style):

  ```
  λ_b(t) = base(vocal) · mood_factor · tod_factor · weather_factor
           + Σ_{recent calls j} α · warmth_b · proximity(b, j) · e^(−(t − t_j)/8s)
  ```

  - **Chorus** emerges when two or more birds with high vocal frequency excite each other within a window. The planner tags the overlapping calls as a chorus event, and the notebook can pick it up.
  - **Call-response.** After bird j calls, bird b answers with probability proportional to warmth_b · proximity. The answer is scheduled 0.6–3 s later as a `response` call.
  - **Voice budget.** At most 5 overlapping calls. Excess ambient calls slip by a few hundred ms. Recognizability depends on this.
- **Newcomer visits** (§5.14): brief visits to back-zone slots, 1–3 per day while an offer is open.
- **Sound-directed attention.** Head-tilt toward sounds is client choreography, driven by the audio event bus, and has no server state.

### 5.10 Bird-to-bird interaction summary

The social system consists of calls prompting responses, wary spreading through alarms, warm birds perching near each other, and choruses from mutual excitation. The server plans all of it, so every device and visitor sees the same small social events at the same moments.

### 5.11 Greeting selection

The greeting is computed by the API when serving a snapshot for `open`, `visible` (hidden ≥ 20 s; D-16), or `resume` (a suspend gap), and never for visitors.

1. **Absence.** `absence = now − last presence end` at account level (§5.4). There is no greeting after a settle followed by re-engagement in the same page lifetime.
2. **Pick the greeter.** It's always exactly one primary. For each non-roosting bird (the nightjar is exempt at night; at night one roosting bird may open an eye as a minimal greeting):

   ```
   score_b = 1.2·boldness + 0.8·warmth + mood_mod(m) + 0.6·A_b + 0.35·Gumbel()
   ```

   `mood_mod`: alert/curious +0.3, content +0.1, drowsy −0.3, wary −0.5. The primary greeter is the argmax. The noise term means the bolder bird usually greets first, but not always. That makes "pip greeted before wren today, first time this week" a real, occasional observation.
3. **Form**, from absence and the bird's character. Absence sets intensity continuously:
   - under 5 min: a glance up from the current activity;
   - minutes to hours: a head-tilt, or a quiet two-note call;
   - hours to a day and a half: a tilt and a step toward the front, or a call;
   - longer: a re-orientation, where the bird comes to the front perch with a longer call and a second bird may answer.

   Each bird has a stable greeting disposition, a weighting over forms derived from its signature seed and traits. So *the same bird greets in its own recognizable way*, while different birds greet differently.
4. **Secondary greeters.** Zero, one, or rarely two. Their probability rises with absence, warmth, and A, and falls with wary or drowsy mood. They are staggered after the primary by `U(0.7 s, 2.5 s)` plus jitter, and never fire in unison. A warier bird may not greet at all that day.
5. **Realization.** The API picks a fresh seed for gaze angle, head-tilt amount, step distance, call motif selection and pitch contour, and timing. The start time is chosen so the notice begins 0.3–1.4 s after the first bird is visible.
6. **Never identical twice.** Compute a fingerprint from the form skeleton plus quantized realization parameters. Re-roll, up to 5 times, if it collides with any of the bird's last 50 greetings. A CI property test generates 100k greetings per synthetic bird and asserts zero identical realizations and a minimum perceptual distance between consecutive greetings.
7. **Recording.** Insert a `greeting` event with the descriptor. The client confirms with `greeting_seen` when it plays.

The snapshot's `greeting` field is a choreography descriptor: bird, form, seed, start offset, and the `response` of any secondary greeter. It is never text. **Nothing textual accompanies the greeting.**

### 5.12 Offers: resolution, reactions, cooldowns

- **Synchronous resolution** on `POST /v1/aviary/offers`, with engine code run in the API:
  1. Take a per-aviary advisory lock (serializing offers from multiple devices) and read the birds plus any resolved offer events since the last tick.
  2. **Candidates** are birds that are not roosting.
  3. **Reaction per candidate** is sampled from curiosity × mood × proximity × A:
     - **Seed:** a curious or content bird approaches and takes it. A wary bird waits 5–30 s, then comes near. A drowsy bird watches, or doesn't approach at all.
     - **Song fragment:** a bird may join in (its call timed to the fragment's phrase gaps, pitch-matched within its register), go quiet for 20–40 s, or call against it. This is shaped by vocal frequency and mood.
     - **Still pool:** birds drink, bathe, or watch. The pool stays as a scene object for about 10 min.
  4. **Credit and cooldown.**
     - Birds on cooldown still react visibly with glances and small tilts, but get no drift credit.
     - **An accepting bird** gets `offer_accepted` credit and a **4-minute cooldown** (configurable 3–5).
     - **Birds that are near** get `offer_near` credit and the same cooldown.
     - The daily caps from §5.5 apply.
  5. Insert the `offer` event with its outcome and return `{object, reactions:[{bird, reaction, delay_ms, seed}]}`.
- **What the client sees.** The item appears immediately when the user offers it (the item itself is optimistic). Birds react when the response arrives, 100–300 ms later, which reads naturally as the bird noticing.
  - The offer affordance is **never disabled** and never shows a countdown. Button-mashing just produces birds that watch.
  - If the POST fails, the item still appears, the birds glance, and nothing is credited. There's no error unless the failure persists (§6.6).
- **Other devices** see scene objects such as the still pool, and the post-offer mood, on their next snapshot.
- **Song fragments** are note sequences in a library of 8, synthesized in a soft whistled timbre that sits outside every bird's signature space. They are never audio files (H4).

### 5.13 Settle

- **Where it lives.** It's in the gesture tray (D-01), and there's a keyboard shortcut (§10.4).
- **What happens.**
  - Lighting transitions toward dusk over 4.5 s (D-19 covers night).
  - The mix ducks by 6 dB, and scheduled calls are thinned locally.
  - One bird acknowledges the goodbye: a glance or a single low call, using a client-side seed.
  - Narration: "the light warms toward evening. the calls grow quiet."
- **Undo.** Any click, tap, or key in the aviary within **5 s** of triggering reverses the lighting shift and the mix. It is exempt from listen-in handling during those 5 s.
- **Commit.**
  - The client holds the `settle` event until the undo window closes, then posts it. On `pagehide` it goes by beacon.
  - Presence ends at the trigger time.
  - The tick applies the settle mood-quieting modifier. Settle has **no** drift effect beyond ending the presence window.
- **After commit.** The aviary stays settled: that session keeps the evening lighting until the tab closes or the user re-engages. Re-engaging means a click or tap in the aviary, a top-bar action, or a key interaction with a bird. Pointer movement alone doesn't count.
- **Re-engaging** eases the lighting back to the current local time over 3 s and resumes presence tracking. There's no greeting, because the user never left.
- **Scope.** Settled lighting is session-local, so other devices and visitors are unaffected (D-08).
- **Closing the tab without settling is identical at the engine level.** There is no recovery surface and no reminder.

### 5.14 Adoption: starters, newcomers, cap, voice distinctness

- **Genesis** at account creation:
  - The engine picks two diurnal species from the pool of six, preferring contrasting silhouettes and call registers. The nightjar is never a starter.
  - It seeds traits (§5.5) and generates immutable voice signatures and plumage.
  - It creates the aviary, which starts ticking immediately.
  - The user does not choose from a catalog.
- **Starter naming.** A short step presents the two arrivals as birds that arrived: silhouettes, naturalist copy, and pre-filled suggested names from a curated list. Names are editable and can be skipped by accepting the defaults.
- **Arrival.** The aviary starts as the quiet field (§7.2). The first bird flies in softly to its starting perch, and the second follows a few seconds later. `arrived_at` makes this a once-per-account moment. After it, the user never sees an empty aviary again.
- **Newcomers (third bird and beyond), by aviary age only.**
  - Schedule: day 90 (3rd), 150 (4th), 240 (5th), 330 (6th), 450 (7th), computed as `next_newcomer_at = max(schedule[n], last_adoption + 30 d)`.
  - Visit count, interaction volume, and payment are **never** inputs. A test asserts that the newcomer schedule is invariant to all event histories.
- **Newcomer presentation.** This is noticing, not announcing.
  - When an offer opens, a newcomer of a species not yet present (duplicates are allowed once all six are present) visits the back perch briefly, one to three times a day.
  - It calls rarely and quietly, has no personality drift, and doesn't count toward the cap.
  - The notebook may note it: "a small finch has been visiting the back perch these last mornings."
  - Listening in on the newcomer opens a small naming card anchored under the top bar, not in the scene, in naturalist voice. It shows a suggested name and two choices: *let it stay* (it becomes a bird with fresh seeds and its own identity) or *let it go on its way* (the next newcomer comes at the next age step).
  - Ignoring it is fine. It keeps visiting without pressure, with no expiry, countdown, or reminder.
  - Pending offers never stack.
- **Cap.** No newcomers are generated once there are seven birds. The global `max_birds` flag (§15.4) can hold the cap lower during rollout. The engine hard-codes 7 as the absolute maximum.
- **Voice distinctness.** A signature is a vector over base pitch, register span, characteristic interval set, rhythmic template, timbre partial profile, trill rate, and a signature tag motif. A new bird's signature is chosen from 200 samples to maximize the minimum perceptual distance to the existing birds and any visiting newcomer, subject to a floor `d_min` validated by listening tests (§8.4). Same-species birds must still clear `d_min`.
- **Identity continuity.** `bird_id` and `voice_signature` never change. Species art revisions (`species_rev`) are pinned per bird and may only be applied through identity-preserving migrations: same silhouette family and same palette anchors, reviewed by design. No migration regenerates a bird.

---

## 6. Sync model

### 6.1 Principle

There is one canonical aviary per account, and one writer: the tick, plus the engine-hosted genesis and adoption paths, which run under the tick role. Clients render snapshots and append events. Multi-device sync is therefore a property of the architecture: two devices read the same record. There is no client-to-client sync, no merge, and no eventual-consistency reconciliation of personality.

### 6.2 Why device concurrency can't lose drift

- **Drift inputs are commutative, idempotent aggregates:**
  - presence is a union of intervals;
  - listen-in is a per-bird union of intervals;
  - offers are credits that were serialized and resolved on the server.

  The order in which devices submit doesn't matter, duplicates are no-ops (unique `event_id`), and overlap is deduplicated by the union.
- **Traits change only as server-computed deltas** applied to the stored vector, inside the tick's transaction. There is no API by which any client submits a trait value. This is the "no last-write-wins" rule, enforced by DB grants.
- **Exactly-once consumption.** The tick advances `event_cursor` in the same transaction that applies the deltas.

### 6.3 Event ordering that is actually correct

A global `bigserial` isn't enough, because values are assigned at insert, not commit. A tick could read seq 105 committed before seq 104, advance its cursor, and skip 104 forever.

The fix: every event insert does `UPDATE aviary_event_heads SET next_seq = next_seq + 1 WHERE aviary_id = $1 RETURNING next_seq` inside the insert's transaction. That row lock serializes inserts **per aviary** until commit, so for each aviary, sequence order equals commit order. Per-aviary event rates are a few per minute, so contention is negligible. Different aviaries don't contend with each other.

### 6.4 Snapshots, projection, and the intent timeline

- **Snapshot = canonical state at the last tick ⊕ projection.** The projection is a pure, read-only fold of events resolved since that tick: offer outcomes, scene objects, and the current greeting. It is never persisted as canonical. The next tick persists the real effects.
- **Timeline commit horizon.** Each tick appends intents out to `now + 5 min`. Intents starting before `now + 45 s` are immutable. That way, a call a client has already scheduled into its audio lookahead, or a flight it has started, is never contradicted by the next snapshot. The one exception is a user-initiated reaction (offer, greeting), which may preempt that bird's committed ambient intents. The projection marks the preemption.
- **Interpolation.** A bird's position between intents comes from the choreographer's flight arcs and pose blending. There's no teleporting. Mood changes cross-fade the idle style over 20–40 s.
- **Versioning.** A client keeps the snapshot with the highest `(tick_seq, projection_seq)` and discards late, out-of-order responses.

### 6.5 Pull cadence

| Trigger | Action |
|---|---|
| Page load | Snapshot is inlined by the edge, or fetched via the SW navigation preload (§7.1) |
| `visibilitychange` → visible | Fetch with `reason=visible`. Greeting if hidden ≥ 20 s. |
| Long frame gap (rAF delta > 2 s while visible), Page Lifecycle `resume`, or `pageshow` with `persisted` (bfcache) | Fetch with `reason=resume` |
| Keepalive while visible | Every 60 s ± 10 s jitter, with `If-None-Match` (304 is cheap) |
| After an offer response | No fetch needed: the response carries the choreography |
| Hidden | Nothing. Rendering and audio stop. The server keeps ticking. |

**Clock offset.** The client keeps an EWMA of `server_now − (t_send + t_recv)/2` over the last 8 responses and rejects outliers with RTT above 2 s. All timeline times are in server time.

### 6.6 Failure and conflict handling

| Case | Behavior |
|---|---|
| Transient network failure | Keep rendering from the timeline. Past its horizon, the client's *ambient continuation mode* keeps birds at their current perches with local seeded idle motion and local ambient calls, so nothing ever freezes. Events queue in memory and IndexedDB and flush with backoff. Presence older than 10 min is dropped on the server as unverifiable. |
| Persistent failure (> 2 min) or initial load failure (> 8 s) | A system-voice line in the top-bar status region: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." The scene stays as it is. Never a spinner. |
| Session expired or revoked mid-write | "Your session timed out. Sign in again to keep watching." The queue is kept. After sign-in, events still inside the lateness window flush and the rest are dropped silently. |
| Magic link replayed, expired, or already used | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| Two devices renaming the same bird | `If-Match` version check. The loser gets a 409, and the settings panel shows "This name was changed on another device." along with the current name. Names are user-owned, not personality, so field-level compare-and-set is enough. |
| Settings edits from two devices | Per-field merge patch. The last write wins **per field**. Settings are preferences, not simulation state. |
| Two tick workers on the same aviary (lease anomaly) | Row lock plus the `version` check. One commits and the other aborts. An aggregate counter tracks this. |
| Snapshot cache stale or unavailable | The API reads from sim_db (primary or a replica with bounded lag) and rebuilds the cache. |
| Server outage | Clients go into ambient continuation. After recovery, catch-up ticks integrate the gap (§5.2), so the aviary "was running" through the outage as far as the model is concerned. |

### 6.7 Protecting the personality vector (the worst failure)

- A synchronous standby, continuous WAL archiving, and PITR (30 days).
- Daily `bird_trait_daily` integrity snapshots, which make it possible to restore one bird without a full PITR.
- The DB trigger that prevents decreases (§3.3) and the immutable-column grants.
- A **nightly integrity job**: every bird row has traits in range and not below yesterday's snapshot, its signature is present and unchanged, its `bird_id` still exists, and the bird count equals the number of adoptions. Violations page on-call as aggregate counts. Investigating a specific aviary requires a break-glass procedure.
- An audited repair procedure: `repair_mode` restores from the snapshot table under the break-glass role, with a written incident record. Repair is data recovery, not engine drift.
- No migration may rebuild birds. The migration review checklist includes an identity-preservation check.

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence and the first frame (time to first bird)

**Cold path** (no service worker yet):

1. The edge receives the navigation. It immediately sends **103 Early Hints** preloading `scene-core.js` (≤ 60 KB gzipped, immutable, cached at the CDN), then starts streaming the HTML shell (≤ 14 KB gzipped, fitting in the first congestion window). The shell contains:
   - inline critical CSS;
   - a ~2 KB inline boot script that detects features, computes the local-time sky gradient, and paints the **quiet field** before any data arrives;
   - the top bar as static HTML with an inline SVG sprite, using system fonts only, so there are no web fonts on the critical path.
2. In parallel, the edge fetches `/v1/aviary/snapshot?reason=open` from origin, passing the session cookie through. The origin serves it from Redis plus the projection, with p95 ≤ 60 ms server time. The edge streams it into the page as `<script type="application/json" id="snap">`. It is data, not executable, and is allowed by CSP.
3. `scene-core` holds the choreographer, the Canvas2D renderer, and the rigs for all six species (~4–6 KB each). It parses the snapshot, evaluates each bird's pose at `server_now + elapsed` (mid-preen, mid-flight, mid-call), and draws the first frame with birds at full opacity, mid-action. Then it calls `performance.mark('first-bird')`.
4. After first paint, in `requestIdleCallback`, it loads the audio engine and worklet, the voice runtime, and prefetches the panels. Then it registers the SW.

**Warm path** (SW installed, which is most visits for a product people return to daily):

- The SW serves the shell and `scene-core` from cache, and uses **navigation preload** to start the snapshot request in parallel with SW startup.
- If the cached last snapshot's timeline still covers `now` (within 5 min), it can render birds before the network responds, then reconcile.
- Expected first bird is about 200–350 ms on the reference mid-tier phone over 4G.

**If the snapshot arrives late** (slow network, cold cache), the quiet field holds: a soft local-time sky with one or two faint motion cues (a leaf drifting through, a gentle shimmer of light). Birds then resolve **already in motion**, with a soft dissolve of at most 300 ms. There's no scale, no bounce, and no wake-up pose. Flights that are in progress on the timeline simply continue into frame. **There is no spinner anywhere in the product, including system panels.** System panels prefetch, and if something still takes over 800 ms, they show the plain word "Loading…" in system voice.

**What the first frame must never contain:** an entry animation, a fade from a static image, a "ready" pop, a welcome text, or a scene-root opacity transition. A visual CI test captures the first painted frame via CDP screencast and asserts two things: every bird is at a non-rest phase of an activity, and the scene root has no CSS transitions.

### 7.2 Special states

- **Empty aviary.** This happens only between naming and the first arrival. It is the quiet field, then a soft fly-in for each starter (§5.14), and it happens once per account.
- **Unsupported browser.** The inline boot detects missing capabilities: ES2022 modules, Canvas2D, `Intl.DateTimeFormat` time zone support, `fetch` streaming, and `IntersectionObserver`. It then renders a static, system-voice page: "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge." Missing WebAudio is **not** a reason to block (§8.7).
- **Signed out.** The sign-in page (system voice), with no aviary rendering.
- **Pending deletion.** The aviary renders normally, plus a persistent system-voice line in the top-bar status region: "Your account is scheduled for deletion on 14 October. [I changed my mind]."

### 7.3 Scene composition and layers

- **World space.** Height is 1000 units. The scene aspect is clamped to [1.0, 2.4]. Perch slots are defined in normalized coordinates per zone. Front slots sit lower and render larger; back slots sit higher, smaller, and slightly desaturated.
- **Layers, back to front:**
  1. **Sky canvas.** A gradient and light for the current lighting keyframe. Redrawn when the lighting bucket changes (about every 20–30 s, or per frame during settle and weather transitions).
  2. **Background foliage canvas.** Cached bitmaps, re-tinted per lighting bucket. Gentle sway comes from CSS transforms on the compositor, not from repaints. This is the "subtle parallax": depth-differentiated sway amplitudes, with the far layer barely moving. It is not a layered show-off illustration.
  3. **Main canvas, redrawn every frame.** Perches, birds, scene objects (seed, still pool), and mid-depth weather.
  4. **Foreground ornaments.** Occasional passing foreground branches or leaves, drifting leaves and feathers, near rain streaks. These are pooled particles and are drawn into the main canvas to avoid another full-screen layer.
- **DOM overlays** above the canvases: caption layer, bird focus proxies, the visually hidden narration live region, and the top bar. All DOM lives outside the scene art. The scene itself carries **no** buttons, badges, tooltips, labels, or overlay icons.
- **Palette.** Calm naturalist tokens from the design system: soft blues and greens, warm browns, muted ochres. No saturated accent colors. All tokens are imported from the design-system package.

### 7.4 Bird rigs and idle micro-motion

- **Rigs.** Each species is a parametric 2D vector rig: body, head, beak, eye and lid, wings (folded, half, open), tail, and legs. Each part is a cached `Path2D` with a pivot, drawn with a transform hierarchy.
  - Plumage colors come from the snapshot's derived palette and the lighting.
  - Feather-detail patterns are pre-rendered per detail level.
  - This costs about 12 fills per bird, so all seven birds take well under 1 ms of the frame.
- **Actions** are procedural curves over pose parameters: breathing (0.5–1% body scale at 0.2–0.35 Hz), preen sequences, scanning saccades with holds, head tilts toward sound sources, weight-shift shuffles, fluffing, blinks every 3–8 s, occasional tail flicks, and flight (flap cycles along Bézier arcs with body pitch).
- **Mood shapes idle style.** A wary bird scans more and holds further back. A content bird preens. A curious bird tilts and follows passing leaves. A drowsy bird sits low and fluffed. Roosting birds have their eyes closed.
- **Anti-strobe rules.** All procedural noise is band-limited below 2 Hz except brief blinks and flicks. Motion envelopes are eased with minimum durations. Phases are randomized per bird so nothing moves in lockstep. A lint on animation curves rejects periodic motion above 3 Hz and sub-100 ms toggles.
- **Pose library constraint.** There are no poses expressing distress, sickness, hunger, or sadness (§1.2).

### 7.5 Transitions and time

- **Perch change:** a flight arc lasting the intent duration.
- **Mood change:** style parameters blend over 20–40 s.
- **Lighting:** continuous keyframes (night, pre-dawn, sunrise, morning, midday, afternoon, golden hour, dusk, night) with a gradual morning warm-up, midday at peak brightness, a warmer evening, and a dim night.
- **Weather:** fades in and out over 30–60 s. Rain adds soft streaks, a slight darkening, and a procedural rain bed in the audio. Wind adds a foliage ripple and a few extra leaves.
- **Offers:** the seed drops softly to the front rail, the pool settles into the front of the scene, and the song fragment is audio plus a faint shimmer.
- **The render loop:** a single `requestAnimationFrame` loop. Time is `performance.now()` mapped to server time. There are no per-frame allocations: bounds, particles, and poses live in preallocated typed arrays and pools.

### 7.6 Responsive layout

- **Portrait phones.** The scene fits the width. The perch band stays horizontal, with extra sky above and foliage below. It compresses horizontally without cropping.
- **Bird scale.** `min(heightScale, widthScale)`, where `widthScale` ensures seven birds plus a newcomer fit at a 320 CSS px minimum width with a minimum 44 px touch target.
- **Wide screens.** Perches spread apart up to aspect 2.4. Past that, the foliage continues at the margins.
- **Guarantee.** Every perch slot and every flight path stays inside the visible safe area at all sizes. A layout property test samples viewports from 320×480 to 3840×1600 in both orientations and checks that no bird's bounds leave the viewport.
- **On resize,** the layout is recomputed and the cached layers are re-rasterized after a 150 ms debounce. Birds snap to their new slot positions without animating.

### 7.7 Top bar

- **Contents.** Exactly four icon buttons, in this order: **offer** (opens the gesture tray; D-01), **field notebook**, **accessibility settings**, **account/settings**.
- **Placement.** It overlays a strip of sky where no perches sit, so fading it reveals scene rather than an empty frame.
- **Fade.**
  - After 4 s of cursor stillness it fades over 1.2 s to 10% opacity. Any pointer movement or keyboard activity returns it over 200 ms.
  - It never fades while focus is inside it or a panel is open.
  - On touch, see D-20.
  - The accessibility settings offer "Keep the top bar visible", and `prefers-contrast: more` raises the faded floor to 60%.
- **No indicators, ever.** No unread dot on the notebook, no badge on settings when a visit happens, no "new" labels. The document title is always "Pocket Aviary" and the favicon never changes.
- **Status region.** A system-voice status line exists only for errors and account-state notices (§6.6, §7.2). It is empty otherwise.

### 7.8 Panels

- The notebook, settings, and accessibility panels open as side sheets on desktop and bottom sheets on mobile. The aviary stays visible and audible around them.
- Focus is trapped inside, Escape closes the panel, and focus returns to the button that opened it.
- The notebook list is **virtualized**: only visible rows plus a small overscan are mounted. Page data sits in a bounded LRU of 10 pages, so scrolled-out entries hold no references (the PRD's memory rule).
- The gesture tray is a small popover with arrow-key navigation, Enter to choose, and Escape to close.

### 7.9 Reduced-motion rendering (a designed surface)

- **When it activates.** When `prefers-reduced-motion: reduce` is set (watched live), or when the user sets "Motion: reduced" in accessibility settings. The options are *Follow system*, *Reduced*, and *Full*.
- **Poses instead of motion.** Each species has a still-pose library of 10–14 poses: perched alert, three preen poses, fluffed, tilt left and right, looking back, low-sitting drowsy, roosting, and near-offer poses. The `choreo` package's reduced-motion sequencer maps the same timeline activities to pose sequences. Each pose is held for 4–10 s, with **slow cross-fades** of 1.5–2.5 s between them.
- **Flights** become cross-fades between perches: out at A and in at B, overlapping, over about 1.8 s.
- **Removed:** ambient leaf and feather drift, foliage sway, and particle rain. Rain becomes a faint static texture plus a light change, cross-faded.
- **Kept, slowed further:** day, evening, and night color shifts, and the settle light shift, which becomes 6 s.
- **Still happening:** greetings (a pose change plus the call), offer reactions (a cross-fade into the near-offer pose), full-quality calls or captions, drift, mood, and notebook.
- **Top bar:** the fade becomes a simple opacity change with no movement. Focus rings don't animate.
- **Enforcement.** Every animation descriptor type in `choreo` has a required `reducedMotion` field (pose sequence, cross-fade, or `none-static`), so the type checker makes it impossible to add an animation without designing its reduced-motion variant.
- **Art direction** owns the look of this mode with its own review. It must read as calmer and deliberate, never as broken.

### 7.10 Hidden tabs, suspend, and bfcache

- **Hidden:** the rAF loop stops, which the browser enforces anyway, and our loop also exits explicitly. Audio fades over 0.8 s and the AudioContext is suspended. Presence is flushed. Heartbeats stop.
- **Visible again:** the client renders immediately from the cached timeline if it still covers `now`. If not, it shows last-known perches in idle poses and fetches the snapshot, then cross-dissolves (≤ 400 ms) any bird whose perch changed while the user was away, instead of flying it back. The greeting is played when the snapshot arrives, within the 1–2 s window.
- **Unfocused but visible:** keep rendering at 60 fps. Keep audio. No presence (D-15).

### 7.11 Input and hit testing

- Pointer events on the scene container are hit-tested against the bird bounds computed in the last render pass. Touch targets are at least 44×44 CSS px.
- Clicking or tapping a bird toggles listen-in on it. Clicking empty space disengages listen-in. During the settle undo window, any click means undo.
- **Focus proxies.** Each bird has a transparent DOM `<button>` whose position updates at about 10 Hz via `transform`, with no layout thrash. The **visible focus ring is drawn in the canvas** so it tracks the bird exactly. It is a double ring, light inner and dark outer, readable against both bright and dim scenes (≥ 3:1 against any background).

---

## 8. Audio pipeline

### 8.1 Graph

```
AudioWorkletNode "syrinx" (outputs: 7 bird buses + newcomer + offer voice + ambient bed)
  bird bus i ─► Gain(level_i) ─► BiquadLowpass(distance/listen-in) ─► StereoPanner(x_i) ─┬─► dry bus
                                                                                         └─► reverb send
  offer voice ─► Gain ─► Panner ─► dry/reverb
  ambient bed (procedural wind/leaf rustle, rain bed) ─► Gain ─► dry bus
reverb: ConvolverNode with procedurally generated impulse response (generated once at init)
dry + wet ─► gentle Compressor/Limiter ─► Master Gain (volume/mute) ─► destination
```

- **The worklet** is about 12 KB. It preallocates a fixed pool of 16 voices and all its buffers, and **allocates nothing in `process()`**. Call scores arrive as compact messages over the port, which gives predictable CPU and flat memory.
- **Each voice** can produce:
  - a sum of 1–4 harmonics with the signature's timbre ratios;
  - a per-note pitch contour (rise, fall, arc, sweep), with FM vibrato and AM tremolo;
  - a noise component through a band-pass, for chips and buzzes;
  - an ADSR envelope;
  - an optional second independent voice, for species with a "two-voice" syrinx.
- **Panning, distance, and the listen-in filter** use native nodes, with parameters updated at about 10 Hz from bird positions via `setTargetAtTime`.
- **Safari's Audio Session API**, where available, is set to `navigator.audioSession.type = 'ambient'`. The aviary then mixes with the user's own music and podcasts instead of interrupting them, and respects the iOS silent switch.

### 8.2 Call grammar runtime (`packages/callgrammar`)

- **Input:** the bird signature, mood and style, the call type (ambient, response, greeting, alarm, chorus-join, offer-response), and a seed.
- **Grammar:** each species has weighted rewrite rules: `Call → Intro? Phrase+ Coda?`, `Phrase → Motif (Gap Motif)*`. Motifs are built from primitives: note (contour), trill (rate, count), chip, buzz, and slur.
- **Signature invariants** make a bird recognizable across mood and drift:
  - fixed timbre profile;
  - base pitch within ±1 semitone across all moods;
  - a characteristic interval set;
  - a rhythmic template;
  - the **signature tag motif**, present in at least 70% of calls.
- **What mood modulates:** tempo (±15%), amplitude, motif selection weights, repetition count, and phrase length. For example, a drowsy call is slower and softer, and an alert call is crisper. Drift in vocal frequency changes **how often** a bird calls (server-side scheduling), not its voice.
- **Output:**
  - a `CallScore`: timed note events with synthesis parameters;
  - a `CaptionDescriptor`: note count, contour summary, texture, grouping and pauses, register, intensity, and perch zone.
- **Variation.** Seeds ensure every call is realized freshly. The engine keeps a 64-entry recent-score hash per bird and re-rolls on an exact collision. The test "no identical CallScore within 10,000 calls of one bird" runs in CI.

### 8.3 Scheduling

- **Ambient, response, and chorus calls** come from the server timeline at exact server times, converted to AudioContext time through the clock offset. They are scheduled with a 200 ms lookahead by a timer that also runs in background-throttled conditions. Throttling doesn't matter much, since audio suspends when the tab is hidden.
- **Reactive calls** are generated on the client from server-issued seeds: greetings, offer responses, and the occasional soft call a listened-in bird makes in response (probability by warmth and mood).
- **Head-tilt events** go to the choreographer's sound bus for curious birds (§5.9).

### 8.4 Chorus mixing and the recognizability gate

- **Mixing:**
  - bird levels follow distance by zone: front 0 dB, middle −3 dB, back −6 dB with a lowpass around 6 kHz and more reverb send;
  - panning follows x position;
  - the voice budget is 5 (§5.9);
  - the master limiter prevents clipping when calls overlap.

  Two procedural calls mixed at runtime make a real chorus, with none of the phase-cancelling artifacts of stacked loops.
- **Loudness.** A gentle, consistent target (around −28 LUFS integrated for the ambient scene) with a default volume that is moderate, never startling. When audio unlocks it fades in over 1.5 s (D-04).
- **Recognizability gate.** This is what justifies the cap of seven, and it is a launch gate:
  - **Automated.** Build random 7-bird aviaries, including same-species pairs. For each bird, render 200 calls across every mood and trait levels 0.2–0.9 with OfflineAudioContext. Extract features (MFCC statistics, pitch contour, rhythm). Nearest-centroid classification must reach ≥ 90% per-bird accuracy, and ≥ 95% for aviaries of five birds or fewer.
  - **Human.** A listening panel runs ABX and identification tests after 10 minutes of familiarization. Participants must identify the calling bird in 7-bird aviaries with ≥ 80% accuracy, and must recognize a bird across different moods.
  - **Failure policy.** If we fail at seven, we improve the signature design or `d_min`. We don't quietly lower the cap. Lowering it would need a product decision.
- **Uncanniness review** runs with the sound designer every milestone: no "ringtone" pure sines, natural micro-timing jitter, onset transients, and no quantized-sounding rhythms.

### 8.5 Listen-in mix, engage and decay

- **Engage:** the focused bird goes to +4 dB and slightly drier (less reverb, so it sounds closer). The others go to −9 dB relative to their ambient level and get a lowpass at about 2.5 kHz. They are **never below −15 dB, and never muted**.
- **Ramps:** `setTargetAtTime` with τ = 0.7 s, reaching about 95% in 2.1 s, in both directions. There are no hard cuts, so it feels like listening, not switching channels.
- **Disengage:** clicking the focused bird again, focusing or clicking another bird (which moves listen-in to it, with both ramps overlapping), clicking empty space, Escape, or moving keyboard focus out of the scene. The mix returns to ambient with the same slow decay.
- **Other effects:**
  - the ambient bed ducks 2 dB during listen-in;
  - a `listen_in` interval is recorded for drift (§4.4);
  - listen-in engages no visual chrome — no outline, highlight, or label on the bird. Keyboard users see only the focus ring.

### 8.6 Autoplay, mute, and lifecycle

- **Autoplay:** see D-04. There is **no** prompt to enable sound. When the context resumes after a gesture, audio joins the chorus that is already in progress.
- **Sound controls** live in accessibility settings: sound on/off and a volume slider. There is no audio icon in the top bar (the four-icon rule). Mute ramps the master to 0 over 0.5 s, then suspends the context to save CPU. Captions are unaffected.
- **Interruptions.** Handle `statechange` to `interrupted` (iOS calls, other apps) and resume automatically when allowed. Hidden tabs fade and suspend (§7.10).

### 8.7 WebAudio fallback

- **Triggers:**
  - no `AudioContext`;
  - context creation or worklet `addModule` fails;
  - the context stays `closed` or errors repeatedly (for example, a hardware fault).
- **Result:** the aviary plays in graceful silence, and **captions turn on by default** for that device. The user can still turn them off.
- The accessibility settings show a system-voice line: "Sound isn't available in this browser. Captions are on."
- There is **no recorded-audio fallback path** at any quality.
- Failures increment an aggregate `audio_context_error{reason}` counter.
- Autoplay suspension is **not** a fallback case. It resolves on the first gesture.

### 8.8 Memory discipline

- One worklet with fixed voice pools.
- Persistent per-bird channel strips, created once and reused for the whole session, and only rebuilt when a bird is adopted.
- The procedural impulse response and noise tables are generated once.
- No per-call node creation, no OfflineAudioContext in the live path, and one AudioContext per page for its whole life.

---

## 9. Voice system (notebook, narration, captions)

### 9.1 One voice package, two registers

`packages/voice` holds phrase banks, a small prose grammar (templates with slots, synonyms, clause ordering, and optional details), and a **register tag** on every user-facing string:

- **`naturalist`** (product surfaces: scene narration, captions, notebook, offer and settle labels, starter and newcomer naming copy): lowercase by default, present tense for scene description, specific, bird-named, with bird verbs (notice, perch, settle, listen in, offer). No exclamation marks. No "you". No announcement framing. Bird names are lowercased in naturalist prose to match the voice, and shown as typed on system surfaces.
- **`system`** (sign-in, account, sessions, export, deletion, visits management, accessibility settings, errors, the unsupported page): sentence case, direct, and plain. It states what happened and what to do, with no naturalist phrasing and no warmth standing in for usefulness. This follows the brief's rule: any surface where the user engages the system *as a system* (money, identity, errors, settings) uses this register.

### 9.2 Voice lint (CI and runtime)

- **Naturalist rules.**
  - No uppercase except inside proper UI labels such as "Field Notebook".
  - No `!`, no "you" or "your", no digits (numbers are spelled out, and only in captions like "three-note").
  - No trait or mood labels presented as labels ("mood:", "boldness"). No state-list syntax ("X at Y").
  - A banned vocabulary list: achievement, unlocked, level, streak, badge, score, points, welcome back, great to see, congrats, creature, pet, animal, character, chirp, song (except "song fragment"), solo, select, highlight, pin, reset, regenerated.
- **System rules.** Sentence case, no naturalist verbs, no banned vocabulary.
- **Corpus test.** `tools/voice-corpus` generates 50k narration lines, 50k captions, and 10k notebook entries from simulated aviaries. The lint must pass with zero violations. A content designer reviews a stratified sample of 500 per release for charm and specificity. Any line that reads generic ("your bird is happier") is a bug.
- **Runtime guard.** Generated strings are linted before display or storage. A failing string is regenerated with a different seed; if it fails again it is dropped. Nothing unlinted is ever shown.

### 9.3 Field notebook generation (server)

- **Happenings** are aviary-side observations emitted by the tick and API. Examples:
  - greeting order changes ("first time this week");
  - firsts (first time on the front perch, first chorus at dawn, first bath in the still pool);
  - chorus events;
  - weather responses (sheltering, fluffing);
  - long quiet stretches in the aviary;
  - behavior shifts detected from **behavior statistics, never trait numbers**, such as "lingers on the front rail more than it used to";
  - nightjar activity;
  - newcomer visits and arrivals.
- **Forbidden detectors, by rule and by review: nothing about the user's behavior.** No visit counts or frequency, session times, "while you were away", streaks, or durations of absence. A detector may use presence *only* for the fact that a greeting happened, which is itself an aviary event, as in the PRD's own example. The detector registry needs voice-owner sign-off for each detector.
- **Salience and sparsity:**
  - each candidate gets `salience = novelty × rarity × birds_involved × kind_weight`;
  - a per-aviary budget (a token bucket) targets **one entry every ~3 days**, with a minimum gap of 20 h and a maximum of 3 per week;
  - a single exceptional event may bypass the gap: an arrival, or a first-ever chorus;
  - the budget depends on time, not sessions, so very active users don't get more entries;
  - during long stretches without presence the rate drops to at most one per week, so a returning user doesn't find a backlog that reads like "here's what you missed".
- **Batch.** The worker runs per aviary at about 21:00 local time. It picks the best candidate of the period if it clears the budget-adjusted threshold, then renders it through the grammar with `phrase_usage` memory so wording doesn't repeat within 90 days. It stores the entry with `observed_at` and the local weekday prefix ("tuesday — …").
- **Read-only and permanent.** There are no edit, delete, or annotate APIs. Scrollback is unlimited.
- **Examples of target output:**
  - "tuesday — pip greeted before wren today, first time this week."
  - "wren is fluffed against the cool air, watching the back perch. low calls only."
  - "rain passed through before dawn. pip sheltered under the back leaves and came out calling."

### 9.4 Screen-reader narration (client)

- **Composer.** It reads the same snapshot and choreography state as the renderer:
  - time of day and light;
  - weather;
  - one to three salient birds (the one being listened to, the greeter, one moving or calling), described by activity, perch, and posture;
  - bird-to-bird moments.

  It writes a short paragraph of running prose in the field-notebook voice. It mixes names and descriptors ("pip, the small grey one") and avoids gendered pronouns by default.
- **Cadence:** one idle update every 30–60 s (randomized), and only if something has materially changed since the last one. Otherwise it waits.
- **Events:** the greeting, offer reactions, settle, listen-in start and end, and newcomer naming get prompt narration within about 1 s of onset, still phrased as observations ("pip looks up from preening and calls twice, softly."), never as state transitions.
- **Queue discipline:**
  - one visually hidden `aria-live="polite"`, `aria-atomic="true"` region;
  - at most one pending message;
  - an event message replaces any queued idle message;
  - idle messages are dropped if the screen reader is still speaking (estimated from length at typical speech rates);
  - there is never more than one message every 8 s except for user-initiated events.
- **User control** in accessibility settings: *Narration: full* (default) / *events only* / *off*, plus *Show narration text*, which displays the same prose visually in an unobtrusive strip below the top bar and meets the contrast rules.
- **Example:** "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."

### 9.5 System copy catalog (initial)

| Key | Text |
|---|---|
| auth.link_sent | "Check your email for a sign-in link. It expires in 15 minutes." |
| auth.link_invalid | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| auth.session_timeout | "Your session timed out. Sign in again to keep watching." |
| aviary.load_error | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| visit.unavailable | "This visit is no longer available." |
| browser.unsupported | "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge." |
| export.requested | "We'll email you a link to your export." |
| deletion.pending | "Your account is scheduled for deletion on {date}." + button "I changed my mind" |
| audio.unavailable | "Sound isn't available in this browser. Captions are on." |
| name.conflict | "This name was changed on another device." |

---

## 10. Accessibility surfaces

### 10.1 Principles in practice

Every accessibility surface gets the actual product: narration in the naturalist voice, a reduced-motion rendering with its own art direction, and captions written from the grammar. Nothing exposes trait values or mood labels to assistive technology (no `aria-label="mood: content"`). All of it ships in v1 (launch gate G-A, §15.3).

### 10.2 Semantics and structure

- **Landmarks:** a `header` for the top bar, and `main` containing a `role="group"` named "aviary" with bird buttons. The document title is static.
- **Bird buttons.** The accessible name is the bird's name. `aria-describedby` points to a short, slowly updated naturalist description ("on the front perch, preening"), refreshed at most every 20 s. `aria-pressed` reflects listen-in, as standard toggle semantics. Visitors' bird elements aren't focusable, since visitors have no interactions, and the narration carries the scene for them.
- **Captions** are `aria-hidden` to avoid double announcements. Calls reach screen readers through the narration.
- **Panels** are `dialog` elements with labelled headings, focus trap, and focus return.
- `role="application"` is not used.

### 10.3 Captions

- **Enabling:** opt-in from accessibility settings. They're on by default only in the WebAudio fallback.
- **Text** is generated from the `CaptionDescriptor` of the call that was **actually synthesized**, never from stored strings. Examples: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch".
- **Placement:** above the calling bird, clamped to the viewport. They fade in over 150 ms at call onset, hold for the call's length plus 1.2 s, and fade out over 400 ms. At most two are visible, with the oldest yielding. In reduced motion there's no movement, only opacity.
- **Contrast:** text sits on a soft scrim sized to the text. The scrim is tuned per lighting keyframe so the text meets **WCAG AA (4.5:1)** against the worst-case pixels behind it. A visual test checks this at dawn, midday, dusk, night, and in rain.
- **Size:** adjustable (small, medium, large) and responsive to browser zoom.

### 10.4 Keyboard

- **Tab order:** the top bar items (offer, notebook, accessibility, account), then the aviary scene as **one tab stop** with a roving tabindex.
- **Entering the scene** focuses the first bird, the leftmost in the current visual order. After that, focus follows identity: if the focused bird flies elsewhere, focus stays with it.
- **Arrow keys:** Left/Right move to the nearest bird horizontally, and Up/Down move between depth zones. The spatial order is recomputed on each key press.
- **Enter or Space** toggles listen-in (D-03). **Escape** exits listen-in without moving focus. Moving focus to another bird, or Tab out of the scene, disengages it.
- **Shortcuts:** **O** opens the gesture tray and **S** opens it focused on "settle". They work only while focus is inside the app and not in a text field. Accessibility settings can turn them off or remap them (WCAG 2.1.4).
- **Settle undo** accepts any key within 5 s.
- **Focus indicators:** the canvas-drawn double ring for birds (§7.11), and design-system focus rings for the DOM chrome. All are visible against both bright and dim scenes.

### 10.5 Contrast and visual preferences

- All user copy (top bar labels and tooltips on focus, panels, settings, errors, captions, visible narration) meets **WCAG AA**. Design tokens carry pre-verified pairs, and axe-core runs in CI on every DOM surface.
- `prefers-contrast: more` makes panels more opaque, strengthens the caption scrim, and raises the top bar's faded floor. `forced-colors: active` switches DOM chrome to system colors, while the canvas scene is left as is.
- Text resizes to 200% without loss. Targets are ≥ 44 px.

### 10.6 Accessibility settings panel (system voice)

| Setting | Options |
|---|---|
| Motion | Follow system / Reduced / Full |
| Sound | On / Off, plus volume |
| Captions | On / Off, plus size |
| Narration | Full / Events only / Off; Show narration text on/off |
| Top bar | Fade when idle / Keep visible |
| Keyboard shortcuts | On / Off / remap |

These are stored in account settings and resolved per device ("Follow system" is evaluated locally). Visitors keep their own preferences in localStorage only.

### 10.7 Regression prevention

- **CI:**
  - axe-core on all DOM surfaces;
  - Playwright keyboard journeys (reach every control, listen-in, offer, settle, undo, notebook scroll, settings);
  - narration and caption voice lint;
  - reduced-motion visual tests (no transforms or particles present; cross-fades used);
  - the reduced-motion type rule (§7.9);
  - focus-ring contrast checks.
- **Manual, every release candidate:** a screen-reader matrix of VoiceOver (macOS Safari, iOS Safari), NVDA (Firefox, Chrome), JAWS (Chrome, Edge), and TalkBack (Chrome), plus a switch-access pass.
- **Research:** paid sessions with disabled users at alpha and beta, covering blind and low-vision users, vestibular disorders, Deaf and hard-of-hearing users, and motor impairments. The question is whether the surface *feels alive*, not just whether it passes.

---

## 11. Accounts, auth, and privacy

### 11.1 Magic-link sign-in

- **Links.** The token is 256-bit random, stored as a SHA-256 hash, expires in **15 minutes**, and is **invalidated immediately on consumption** (atomic UPDATE). The GET landing page never consumes a token, which defeats link scanners, and it auto-POSTs.
- **Sign-up is sign-in.** The first successful verification creates the account, runs genesis, and goes to starter naming.
- **Rate limits,** keyed on the email blind index in Redis with a TTL: 5 per 15 min and 20 per day, plus an IP-prefix limit of 30 per hour. The response is identical whether or not the account exists. Throttled requests get a system message: "Too many sign-in requests. Try again in a few minutes."
- **Cross-browser.** A link opened in a different browser signs in that browser, as specified. If an in-app email webview is detected, the page adds a system note on how to open the link in the user's browser (R-12).
- **Email delivery** uses a transactional provider with SPF, DKIM, and DMARC, a warm dedicated IP, a secondary provider for failover, and bounce and complaint webhooks feeding aggregate counters.

### 11.2 Sessions

- Each successful sign-in on a device creates a session with an opaque token in an HttpOnly cookie.
- Settings list the sessions by device label and last-active date, and any session can be revoked.
- Revocation is immediate at the origin. The session cache TTL is ≤ 30 s, and revocation also evicts the cache key directly.
- Session lifetime is 90 days sliding and 1 year absolute.
- The session token is rotated on privilege-sensitive actions: email change, deletion, and restore.

### 11.3 Email change

- The request emails a verification link to the **new** address. The old address keeps working until the new one verifies.
- The old address also gets a system-voice security notice.
- On verification, the ciphertext and blind index swap in a single transaction.

### 11.4 Synthetic account ID and PII containment

- `account_id` is a random UUID created at account creation. It is the only account reference in sim_db, logs, metrics labels (only where needed, never in RUM), queues, cache keys, shard keys, and error reports.
- **The email appears only** in `auth_db.accounts.email_ct` and in transient in-memory use when sending mail. It is never a key, never logged, and never in a URL.
- **The blind index** is confined to the auth service and never leaves it.
- **Visitor emails** are encrypted under the host's key and appear only in the visit log UI for the host.
- **Enforcement:**
  - a logger with an **allowlist** of structured fields (unknown fields are dropped, not logged);
  - a CI PII lint that flags `email` identifiers outside the auth package;
  - request and response bodies are never logged for `/events`, `/offers`, names, or auth;
  - error reports scrub all strings from user data (bird names included);
  - a quarterly log-sampling audit.

### 11.5 Export

1. `POST /v1/account/export` enqueues a job.
2. The job assembles the JSON: account settings, birds (id, species, name, adoption date, **current personality vectors**, current moods), and all notebook entries.
3. It encrypts the file at rest in object storage with a 7-day lifecycle.
4. It emails the **verified address** a link that requires a signed-in session for the same account.

The export is never rendered in the product (D-02). It includes a schema version and a short plain-language header saying the file is a copy of the account's aviary data.

### 11.6 Deletion

- **Soft delete.** `POST /v1/account/delete` sets `pending_deletion` and `purge_after = now + 30 d`. From then on:
  - sessions stay valid, and every signed-in page shows the deletion notice with **I changed my mind**, which restores the account immediately;
  - all visit access is suspended, and visitors see "This visit is no longer available.";
  - the simulation keeps ticking, so a restored aviary has kept living.
- **Hard delete (day 30).** The purge job:
  - deletes every row tied to the `account_id` and `aviary_id` in both databases (birds, vectors, events, notebook, happenings, sessions, invitations, visit logs, exports);
  - destroys the account's data key (crypto-shredding any remaining ciphertext);
  - appends the UUID to a **deletion ledger** that is replayed after any backup restore.
- **Consistency with retention.** Backups expire at 30 days and logs at 14, so a hard delete leaves nothing behind in operational stores within the window. Aggregate telemetry has no per-account dimension, so there is nothing to delete there. That is the point.

### 11.7 Privacy boundary as architecture

- **Data classes:**
  - *(a)* per-account interaction and simulation state, which lives in sim_db only;
  - *(b)* identity, which lives in auth_db only;
  - *(c)* aggregate operational telemetry.
- **Rules:**
  - Class (a) is read only by the engine, API, and notebook worker, to drive that account's own aviary.
  - The analytics and observability plane has no credentials for sim_db, and there is no network route to it.
  - No ETL, no warehouse replica, and no ML training on (a) — ever.
  - No aggregates over (a), not even "average drift" or "mood distribution" dashboards.
  - (a) is never shared with third parties, which is why no LLM vendor (D-14) and no third-party analytics or session-replay SDKs.
- **Metric registry.** Every metric and RUM field is declared in a registry reviewed by the privacy owner (§13.5). Introducing a label with a per-account dimension fails CI.
- **Privacy policy.** Account settings link to a plain-text privacy policy that names the aggregate telemetry categories and explicitly excludes per-bird interaction state.

### 11.8 Security baseline

- A strict CSP (hash-based for the inline boot script; no `unsafe-inline`) and Trusted Types.
- HSTS preload; `Permissions-Policy` disabling camera, microphone, geolocation, and payment.
- Cookie hardening; Origin checks.
- Dependency scanning.
- A third-party penetration test before beta, focused on magic links, session fixation and revocation, visitor-pass scoping, and IDOR on bird and notebook endpoints (every query is scoped to the session's aviary).

---

## 12. Visits (read-only, ambient, off by default)

### 12.1 Flow

1. **Invite.** In account settings, which are code-split and never shown at onboarding, the host enters one visitor email and sends. The invitation is `outstanding` with `expires_at = +30 days`. The email is system voice: "{host email} invited you to visit their aviary on Pocket Aviary." It has one link and no marketing.
2. **Claim.** The visitor opens `/visit/{public_id}?t=…`. The landing page doesn't consume the link on GET. JS POSTs `/v1/visit/claim`, which consumes the one-time link token and issues a **visit pass**: an opaque token in an HttpOnly cookie scoped to `/visit/{public_id}`, bound to that browser. The invitation becomes `active`.
3. **View.** The visitor client is the same renderer and audio engine in **visitor mode**:
   - it fetches `/v1/visit/snapshot` (host's canonical state, host's time zone, default ambient mix, current weather and scene objects, no greeting);
   - there's no presence module, no events, no listen-in, no offers, no settle, and no notebook;
   - the top bar shows only accessibility settings, which are local to the visitor.
4. **Duration.** `POST /v1/visit/heartbeat` every 60 s writes `visit_sessions` in auth_db for the host's visit log. It is **never** written to sim_db, and the tick never reads it.
5. **Revocation.** The host revokes from settings. The status becomes `revoked` immediately. At the visitor's next snapshot pull (≤ ~60 s), the API returns 410, and the client fades audio and visuals to the quiet field and shows "This visit is no longer available." A revoked or expired link that was never used shows the same surface. The host gets no confirmation beyond the log.
6. **Expiry and lapse.** Unused invites expire at 30 days and can't be revived. The host can issue a new one. Active passes lapse after 90 days without a visit (D-12).

### 12.2 What's structurally impossible

- Visitor passes are authorized by separate middleware that only allows `/v1/visit/*` read routes. A test enumerates every route and asserts a visitor pass is rejected by all non-visit routes.
- No visitor identity appears in the host's scene: no cursor, avatar, marker, or "someone is here" overlay. The host's view doesn't change during a visit.
- No chat, comments, or annotations exist in any schema.
- Visitor emails are never matched against accounts. There's no friend graph, no suggestions, and no "people you may know".
- No public directory, featuring, rating, or counts. "Most visited" isn't computable, because visit logs are per-host and never aggregated.
- There is no "show-off" rendering path. The visitor snapshot is built from the same builder with fields *removed*, never embellished.

### 12.3 Host-side visit log and notifications

- **The visit log** lives in account settings: visitor email, date, and approximate duration (rounded to 5 min), newest first, plus outstanding invitations with revoke buttons. There's no badge, dot, or count anywhere when a visit happens.
- **Visit notification toggle.** A per-account setting, **off by default**, never offered during onboarding. When on, a system-voice email goes out after a visit ends: "{visitor email} visited your aviary on {date}." At most one per visitor per 24 h. This is the only aviary-related email that can ever be sent, and only when opted in.

---

## 13. Performance budgets and observability

### 13.1 Budgets

| Budget | Target | Gate |
|---|---|---|
| Initial JS at first paint (gzipped) | PRD cap **< 2 MB**. Internal target ≤ 350 KB: inline boot ~2 KB, scene-core ≤ 60 KB, deferred audio/voice ≤ 90 KB. Panels are split and loaded lazily. | `size-limit` in CI warns at 400 KB and fails at 600 KB. The 2 MB cap is never approached. |
| HTML shell | ≤ 14 KB gzipped (first flight) | CI |
| Snapshot | ≤ 8 KB gzipped for 7 birds (typical 2–3 KB) | Contract test |
| Time to first bird | **< 500 ms** from navigation on the reference mid-tier phone over 4G. RUM SLO: p75 ≤ 500 ms. | Lab gate: warm-connection and repeat visits ≤ 500 ms at p75 of 20 runs. Cold first visits are tracked separately and driven down (R-06). |
| Snapshot API | p95 ≤ 60 ms server time; p99 ≤ 150 ms | Alert |
| Idle motion | **60 fps on a five-year-old mid-range laptop for the whole 30-min session**: p95 frame ≤ 16.7 ms, dropped frames ≤ 1%, main-thread script per frame p95 ≤ 5 ms | Nightly device-lab run |
| Memory | **No growth over 30 min**: retained JS heap at min 30 minus min 5 (after forced GC) ≤ 1 MB. Stable detached DOM nodes = 0. AudioNode count constant. Worker count ≤ 1. One AudioContext. | CI (§13.2) |
| Tick | p99 duration **alarm at 5 s** (PRD); target ≤ 250 ms; lag p99 < 90 s | Alert |
| Browser support | Last two major versions of Chrome, Safari (macOS and iOS), Firefox, Edge | CI matrix |

**Reference devices** (pinned in the device lab):

- **Mid-tier phone:** a current-generation mid-range Android (Galaxy A5x class) plus an older iPhone still receiving the latest iOS.
- **Five-year-old laptop:** a 2021 mid-range Windows laptop (Core i5-1135G7 / Iris Xe, 8 GB) running Chrome and Edge, plus a 2020 Intel MacBook Air running Safari.
- **4G profile:** about 9 Mbps down, 85 ms RTT, 1% loss. It's documented and applied identically in lab and synthetic runs.

**The adaptive quality controller** watches frame times. If p95 exceeds budget for 10 s, it reduces things in this order: DPR cap (2 → 1.5), particle count, foliage sway layers, rain density. **Bird motion quality is never degraded first.**

### 13.2 The 30-minute memory test (a real CI job)

- **Nightly:** Playwright with Chromium via CDP, plus a Firefox variant. It runs a **real 30-minute session** against a staging aviary with seven birds. The script covers:
  - idle watching;
  - 20 listen-ins;
  - 10 offers of each kind;
  - settle and undo;
  - 300 notebook rows scrolled;
  - panels opened and closed;
  - weather toggled via a staging override;
  - resize and visibility cycles.

  It takes heap snapshots after forced GC at minutes 5 and 30 and fails on growth over 1 MB, on any growth in detached nodes, or when instrumentation counters show more AudioNodes, canvases, or listeners. Only in a debug build, which is exercised in CI.
- **Per-PR:** a 5-minute version of the same test with the same assertions (short-leak detection).
- **Weekly:** the device lab repeats the run on the reference laptop in Safari, watching process memory from the OS.

### 13.3 What we measure

- **RUM** (aggregate only, sampled at 25%):
  - TTFB, FCP, and **first-bird** time (`performance.mark` plus a paint timing observer);
  - frame-time histogram and long-animation-frame counts;
  - JS error class counts (scrubbed);
  - audio unlock latency and `audio_context_error{reason}`;
  - snapshot fetch latency;
  - event POST failure counts;
  - **anonymized session-duration histogram**. The client buckets its own duration locally and sends one bucket value by beacon at `pagehide`. There are no IDs, and no join is possible.

  Dimensions are limited to browser family and major version, device class, coarse country, and release.
- **Server:**
  - per-route latency and error rates;
  - tick duration (p50, p99), tick lag, catch-up backlog, expedited tick count;
  - event ingest rate, presence rejections (aggregate), anomaly clamp count, integrity job violations;
  - snapshot cache hit rate;
  - email send latency, bounce and complaint rates;
  - magic-link outcome counts (sent, consumed, expired, replayed);
  - export and purge job durations and success;
  - capacity counters (total aviaries ticked, total birds ticked).
- **Synthetic monitoring:** a scheduled fleet running every 10 min from about 6 geographies × 3 device profiles against dedicated synthetic accounts. It measures first-bird, availability, the sign-in path (against a test mailbox), bundle size, and visitor-view availability.
- **SLOs:** availability of snapshot and events at 99.9%, first-bird p75, and tick lag. There are error budgets and burn-rate alerts.

### 13.4 What we deliberately don't measure

- Any per-account or per-bird metric in telemetry: drift, mood, offers, listen-ins, presence per user.
- DAU/MAU by account, retention cohorts, funnels, visit frequency, streak-like distributions, visits per aviary, birds-per-aviary distributions, "most active" anything.
- Session replay, heatmaps, third-party analytics, ad pixels, fingerprinting.
- A/B tests on engagement. We don't experiment on attention. Quality changes are validated in the lab and by consenting research participants.

These aren't oversights. Collecting them would create the data product, and the internal pressure toward engagement features, that the PRD refuses.

### 13.5 Metric registry and review

Every metric, label, and RUM field lives in `infra/telemetry/registry.yaml` with its owner, purpose, and dimensions. CI rejects labels named or typed as account, aviary, bird, session, device, or email identifiers in any aggregate pipeline. The privacy owner approves registry diffs.

### 13.6 Latency levers for distant users

v1 runs a single primary region with a global edge. If RUM shows first-bird p75 above 500 ms in some regions, the lever is regional read replicas of the snapshot cache, fed by the outbox, plus regional API read nodes. Writes stay in the primary region. This is planned but not built for launch.

---

## 14. Quality strategy and guardrails

### 14.1 Engine

- **Property tests** (fast-check):
  - for arbitrary event sequences and gaps, no trait ever decreases;
  - tick results are invariant to how `dt` is split into ticks;
  - duplicate events are no-ops;
  - device order permutations give identical state;
  - the newcomer schedule is invariant to event history;
  - mood has no dependence on absence length.
- **Golden replays:** recorded event streams must reproduce the state bit-for-bit for a given `engine_version`.
- **Calibration harness** (§5.6) on every constant change, with an HTML report archived per release.
- **Greeting and call no-repeat tests** (§5.11, §8.2).

### 14.2 Sync

- **Concurrency tests** run against a real Postgres:
  - 2–4 simulated devices submitting overlapping presence, listen-ins, and offers, with duplicates and out-of-order delivery;
  - chaos (killed tick workers, lease expiry mid-tick, cache outages, clock skew of ±5 min);
  - assertions that the final traits exactly equal the single-writer reference computation.
- **The commit-order sequence test** (§6.3) with interleaved insert transactions.
- **Snapshot monotonicity** and client discard of out-of-order responses.

### 14.3 Presence

The truth-table suite (§5.4) plus a real-hardware check that suspending and resuming a laptop credits nothing for the suspended time.

### 14.4 Rendering and audio

- **Visual regression** at dawn, midday, dusk, night, rain, wind, and settle; for reduced motion; for 2 and 7 birds; and at phone-portrait, laptop, and ultrawide sizes.
- **The first-frame test** (§7.1).
- **The layout containment property test** (§7.6).
- **Audio lab:** OfflineAudioContext renders, the recognizability gate (§8.4), a loudness check, and a clipping check under 5-voice overlap.

### 14.5 Voice and anti-pattern guardrails

- **Voice lint** in CI and at runtime (§9.2).
- **Copy registry.** Every UI string has a register tag, and strings without one fail the build.
- **Banned APIs and patterns (lint):**
  - `Notification`, `PushManager`, `navigator.setAppBadge`, and `document.title` mutations outside boot;
  - toast components (none exist, and a lint blocks introducing one);
  - `setInterval`-driven UI counters;
  - audio asset imports (`.mp3`, `.wav`, `.ogg`, `.m4a`, `.flac` rejected by the bundler plugin).
- **PR template checklist:**
  - Does this announce anything?
  - Does this surface user behavior back to the user?
  - Does this expose a number about a bird?
  - Does this add chrome inside the scene?
  - Does this add an engagement metric?
  - Does it have a reduced-motion variant?
  - Which voice register?

  Any "yes" requires product and design sign-off.

### 14.6 Security and privacy

- Automated DAST on auth flows.
- The pentest before beta.
- Log-field allowlist tests.
- The metric registry lint.
- A network-policy test proving that the analytics plane can't reach sim_db.

---

## 15. Rollout

### 15.1 Phases

| Phase | Audience | Duration | Purpose |
|---|---|---|---|
| **Staging with time acceleration** | Engineering | Continuous from M1 | Staff-only staging aviaries run the engine with a time-acceleration factor, so 90-day newcomers, seven-bird aviaries, and months of drift can be seen before any real user gets there. Acceleration exists only in staging builds and can't be enabled in production. |
| **Internal alpha** | Staff (~50) | 4 weeks | Real-time daily use, a diary study on "does it feel alive / does it ever announce", screen-reader and reduced-motion dogfooding, performance on personal devices. |
| **Closed beta** | Waitlist invitations, ~1,000 → ~5,000 accounts | 8–12 weeks | Load and ops validation. A **consenting research cohort** (a separate, explicit research agreement) provides the only calibration evidence from real users: guided interviews at days 7, 21, and 35 about whether they notice change. The data comes from interviews and self-reports, never telemetry. Disabled-user research sessions. |
| **General availability** | Open sign-up with a capacity guard | — | Sign-up throttled by an admission rate (a waitlist if tick capacity headroom drops below 40%). Quiet launch, no growth-hacking mechanics. |

### 15.2 Launch gates (all must pass for beta; re-verified for GA)

- **G-A (Accessibility):** narration, reduced-motion mode, captions, keyboard navigation, and AA contrast are complete. The screen-reader matrix passes. Disabled-user sessions don't surface any "degraded variant" findings.
- **G-P (Performance):** budgets in §13.1 met in the lab. The 30-minute memory job has been green for 14 consecutive nights.
- **G-E (Engine):** the calibration harness passes all personas, all property tests pass, and the recognizability gate passes at seven birds.
- **G-S (Sync/integrity):** the concurrency and chaos suite is green, the integrity job is live, and PITR restore and single-bird restore have been rehearsed.
- **G-V (Voice):** zero corpus lint violations, and the content designer has signed off on notebook and narration samples.
- **G-X (Privacy/Security):** the metric registry has been reviewed, the pentest has no open high or critical findings, the log audit is clean, and the deletion purge and ledger replay have been rehearsed.
- **G-N (Non-goals audit):** a cross-functional walk-through of every surface against §1.2 and the PR checklist.

### 15.3 Launch sequencing detail

- Visits ship **dark** in alpha and are enabled in beta. They are always off by default per account, and the feature flag only makes the invite UI available.
- The notebook worker runs from alpha. Its budget constants are tuned from the diary study for sparsity.
- **Kill switches** (server flags, no deploy):
  - weather generation;
  - notebook generation;
  - visits (invite creation plus visitor snapshots);
  - expedited ticks;
  - the adaptive quality floor;
  - **drift integration pause**, which freezes drift and leaves attention filters running, for suspected miscalibration. Pausing is safe because it only delays forward drift.

### 15.4 Ramping birds per aviary

The number of birds per aviary is gated first by aviary age: no real aviary can have a third bird until 90 days after its creation. The ramp is explicit:

1. **Launch:** `max_birds = 2` globally. The newcomer engine runs in *shadow* mode: it computes schedules but shows nothing. Seven-bird paths are exercised only in staging acceleration, synthetic monitoring aviaries, and CI.
2. **About day 80 of the earliest beta cohort:** re-run the seven-bird performance and memory tests on the reference devices, plus the recognizability gate. Then set `max_birds = 3`. Beta aviaries start seeing their first newcomer at day 90. We watch aggregate frame-time and audio-error RUM segmented only by *bird-count bucket of the rendering client*: the client reports the number of birds it is rendering as a coarse dimension (2, 3–4, 5–7), with no account link.
3. **Raise the cap** step by step (4 → 5 → 6 → 7) ahead of the schedule dates (days 150, 240, 330, 450 of the earliest cohort). Each step requires the same checks to stay green.
4. **Rollback** lowers `max_birds` for *new* adoptions only. **Existing birds are never removed or hidden** (identity continuity).
5. **Capacity planning** uses the tick worker's own counters (total birds ticked) and expected age cohorts from account creation counts. No per-aviary distributions are needed.

### 15.5 Engine version rollout

- New drift or mood constants ship as a new `engine_version`, rolled out to aviary-hash cohorts at 1%, 10%, 50%, then 100%.
- Before rollout, the calibration harness diff report shows the effect on every persona.
- Rollback returns to the previous constants going forward. Traits already accrued are not reversed. Because of that asymmetry, **rate increases are rolled out more cautiously than decreases** (R-01).

### 15.6 Day-one instrumentation

Everything in §13.3 is live before the first alpha user. Dashboards: first-bird by region and device class, frame health, audio errors, tick health (duration, lag, backlog, clamps, integrity), sync health (event ingest, rejections, CAS conflicts), auth health (email latency, link outcomes), jobs (export, purge, expiry). Alerts: tick p99 > 5 s, tick lag p99 > 90 s, snapshot error rate > 0.5%, integrity violations > 0, email latency p95 > 60 s, first-bird p75 regression > 20% week over week.

### 15.7 Runbooks

Tick backlog or catch-up; snapshot cache loss; primary DB failover; email provider failover; mass magic-link failures; audio regressions after a browser release (pin the problematic path behind a flag); drift anomaly (pause integration, investigate with the harness, repair from the ledger only under break-glass); privacy incident (a telemetry field leak: purge the sink, rotate, post-mortem).

---

## 16. Team, workstreams, milestones

### 16.1 Workstreams (~15 people)

| Workstream | Staffing | Owns |
|---|---|---|
| Engine and simulation | 2 engineers | `engine`, tick workers, harness, notebook detectors |
| Platform, sync, and auth | 2 engineers | API, schemas and grants, event log, snapshot/projection, auth, visits, jobs |
| Scene rendering | 2 engineers + 1 technical artist + 1 illustrator | Choreographer, renderer, rigs, reduced-motion poses, layout |
| Audio | 1 audio/DSP engineer + 1 sound designer | Worklet synth, call grammar, mixing, recognizability lab |
| Voice and content | 1 content designer + (shared) engineer | Voice package, phrase banks, lint, copy catalog |
| Accessibility | 1 a11y engineer + external testers | Narration integration, keyboard, captions, audits |
| SRE and observability | 1 engineer | Edge, infra, telemetry registry, SLOs, synthetic monitoring, device lab |
| Product, design, QA | PM, design lead, 1 QA | Gates, anti-pattern reviews, research |

### 16.2 Milestones

| Milestone | Weeks | Exit criteria |
|---|---|---|
| **M0 Foundations** | 0–3 | Monorepo, CI budgets, protocol package, DB migrations with grants and trigger, engine skeleton with seeded RNG, harness skeleton, art direction for 2 species, first call-grammar prototypes in the audio lab, voice style guide and lint v0 |
| **M1 Vertical slice** | 4–10 | Magic-link sign-in; genesis; the tick with presence → attention → drift and a basic mood model; timeline planner; snapshot plus edge streaming; Canvas scene with 2 species, idle motion, and a first frame mid-action; worklet calls with 2 signatures; presence module with the truth-table suite; greeting v1; top bar with fade; first measurement of first-bird on reference devices |
| **M2 Feature complete** | 11–18 | 6 species, including the nightjar; offers (3 kinds) with cooldowns; listen-in mix; settle and undo; weather and daylight; chorus and call-response; notebook worker; narration; captions; reduced-motion surface; keyboard; accessibility settings; sessions and revocation; email change; export; deletion; visits; newcomer engine (shadow); SW warm path; unsupported page |
| **M3 Hardening** | 19–24 | Gates G-A through G-N green; pentest; staging-acceleration runs of one year of simulated aviaries; runbooks; internal alpha starts at week 19 |
| **Beta** | 25–36 | Research cohort readouts at days 7, 21, and 35; drift constants confirmed or adjusted (only slower, or via a cautious speed-up); capacity plan |
| **GA** | ~37+ | Gates re-verified; admission-controlled open sign-up |
| **Post-GA** | +~6 weeks onward | `max_birds` ramp begins as the earliest beta cohort reaches day 90 (§15.4) |

**Critical path:** the call grammar and recognizability work, and the calibration harness, start in M0. They have the most unknowns and gate the product's core promise.

---

## 17. Risks

| ID | Risk | Likelihood / impact | Mitigation | Detection |
|---|---|---|---|---|
| R-01 | **Drift miscalibration.** Too fast makes it a Tamagotchi and is irreversible without decreasing traits; too slow makes it a screensaver. | Medium / high | A physics-style harness with personas and hard acceptance bands; daily saturation; a low-pass filter; headroom scaling; err slow at launch; cautious rollout of rate increases; a drift pause switch; research-cohort interviews | Harness reports; aggregate anomaly-clamp counter; research sessions (no per-account telemetry by design) |
| R-02 | **Presence inflation or corruption** from a laxer definition creeping in, multi-device double counting, or jiggler tools | Medium / high | A single presence module; truth-table tests; the server-side union across devices; interval validation; daily saturation that limits damage from fake activity | Rejection counters; truth-table CI |
| R-03 | **Lost or corrupted personality** (the worst failure, and silent) | Low / critical | Single writer; DB grants and the decrease trigger; commit-ordered sequences; transactional cursor; PITR plus daily trait snapshots; nightly integrity job; no bird-rebuilding migrations | Integrity job pages; chaos tests |
| R-04 | **Sync edge cases:** out-of-order commits, duplicate delivery, clock skew, tick overlap | Medium / high | Per-aviary event heads; idempotent event IDs; commutative aggregates; version CAS on ticks; timeline commit horizon; client clock offset | Concurrency and chaos suite; CAS-conflict counters |
| R-05 | **Audio uncanniness:** beepy or ringtone timbre, audible repetition, chorus mush, loudness spikes | High / high | A dedicated sound designer; a syrinx-inspired worklet with noise and FM; micro-timing jitter; no-repeat checks; voice budget; limiter; milestone listening panels | Listening panels; recognizability gate; beta interviews |
| R-06 | **Cold first visit misses 500 ms** on real 4G (DNS, TLS, and the origin fetch dominate) | Medium / medium | Early Hints; a 14 KB shell; a tiny scene-core; HTTP/3; SW plus navigation preload for repeat visits (the dominant case); a quiet-field fallback that reads as the aviary catching up; the regional read-cache lever | RUM first-bird split by cold and warm; synthetic geos |
| R-07 | **Autoplay policies** stop "calls already audible" on first load (especially Safari and iOS) | High / medium | Join-in-progress unlock on the first gesture (D-04); no prompt; the ambient audio session category | Aggregate audio-unlock latency |
| R-08 | **Accessibility regressions,** such as narration drifting into state lists, a new animation without a reduced variant, or live-region spam | Medium / high | Voice lint; the required reduced-motion type; queue discipline; release SR matrix; disabled-user research | CI; release audits |
| R-09 | **Gamification or announcement creep** from well-meaning contributors ("just a small toast") | High over time / high | Banned-API and copy lint; PR checklist; the metric registry forbids engagement metrics; the refusals documented as architecture (§1.2) | Code review; the G-N audit each release |
| R-10 | **Privacy leakage:** email or bird names in logs or error reports, or per-account dimensions sneaking into metrics | Medium / high | Field allowlists; PII lint; registry lint; network isolation; scrubbing; quarterly audits | Audits; CI |
| R-11 | **Notebook becomes repetitive or generic** over months | Medium / medium | Large phrase banks; combinatorial grammar; `phrase_usage` memory; salience-based sparsity; content-design review of long-horizon corpora from staging acceleration | Corpus review of simulated one-year aviaries |
| R-12 | **Magic-link friction:** scanners consuming links, in-app webviews, deliverability | Medium / medium | POST-to-consume; webview guidance; dedicated IP and authentication; secondary provider | Link outcome counters; email latency |
| R-13 | **Tick cost at scale** | Low in v1, medium later | Batching; sharding; the dt-correct engine allows a tiered cadence for dormant aviaries later with identical results | Capacity counters; tick lag |
| R-14 | **The export tension** (D-02): users see trait numbers by exporting | Low / medium | The export is not a product surface, is never rendered, and exists for portability; confirm with product and legal | — |
| R-15 | **Browser changes** break Safari audio or canvas behavior | Medium / medium | Last-two-majors CI matrix; beta-channel canaries in synthetic monitoring; flags to route around regressions | Synthetic monitoring on beta channels |
| R-16 | **Time zone edge cases** (DST, travel, devices in different zones) | Medium / low | IANA tz on the server; canonical account zone with hysteresis (D-17); daylight from tz coordinates | Unit tests across DST transitions and hemispheres |
| R-17 | **Recognizability fails at seven birds** | Medium / high | Early audio-lab work (M0); a signature-distance optimizer; same-species tests; the escalation policy (fix the synthesis, don't silently lower the cap) | Recognizability gate |

---

## 18. Items to confirm with product and legal (non-blocking; defaults are chosen)

1. D-02: personality vectors in the export (default: included, never rendered).
2. D-01: settle inside the offer gesture tray, as opposed to a fifth top-bar glyph (default: the tray).
3. D-12: 90-day lapse for active visit passes.
4. D-18: identifying the host in invite emails by their email address.
5. The research-cohort consent and protocol for calibration interviews (the only real-world drift evidence we allow ourselves).
6. The curated default name lists and the six-species roster sign-off from art direction.

---

## 19. Glossary of plan-internal terms

- **Intent timeline:** server-authored, timestamped coarse intents per bird (fly, act, call) covering about 5 minutes ahead. Anything within 45 s is frozen.
- **Choreographer:** the client's pure function from timeline × time to poses and positions. It never mutates state.
- **Projection:** the read-only fold of events resolved since the last tick into the snapshot. It is never persisted as canonical.
- **Attention (a_c, A_b):** per-bird low-passed input state. The a_c values drive drift; A_b modulates expression and decays with absence. It is not a trait.
- **Daily credit:** a saturating per-local-day accumulator for presence and listen-in credit.
- **Happenings:** aviary-side observations that feed the notebook, never about the user.
- **Newcomer:** a visiting bird offered by aviary age. It becomes a bird only when the user welcomes it.
- **CallScore / CaptionDescriptor:** the grammar's synthesis instructions for one call, and the structured description that captions are written from.
- **Quiet field:** the loading and empty-aviary state: a local-time sky with faint motion, never a spinner.
- **Gesture tray:** the popover opened by the offer icon: seed, song fragment, still pool, and settle.
