# Pocket Aviary — v1 Implementation Plan

*wave_002 · plan 001 · planning deliverable only (no product code)*

This plan turns the Pocket Aviary PRD into a build program that a frontier engineering team can execute without coming back with questions. It assumes the reader has read the PRD. It does not restate the PRD. It interprets it: where the PRD states a rule, this plan names the mechanism that enforces the rule and the test that proves it. Where the PRD is silent or contradicts itself, this plan makes a call and logs it in §1.3.

---

## 0. Orientation

### 0.1 The fifteen decisions everything else depends on

1. **The server is the only writer of simulation state, and the database enforces it.** Personality, mood, perch, plan, weather, and notebook tables can be written only by the `sim_writer` Postgres role. The API role has `SELECT` on them and nothing more. "Clients never write personality" is a grant, not a convention.
2. **Drift is monotonic by construction.** Each trait update is `Δv = r · D · (c − v)² / (c − v₀)`, where the drive `D ≥ 0` is a low-pass-filtered attention signal and `c` is a per-bird ceiling. No input can make `Δv` negative. A Postgres trigger rejects any `UPDATE` that lowers a trait, which covers bugs, bad migrations, and bad restores.
3. **"Quieter after absence" is not drift.** A separate, medium-timescale `attunement` state (recency-weighted presence, half-life about 4 days, with a floor) scales only user-directed behavior: how many birds greet and how eagerly they approach the viewer. It never touches wariness, plumage, call rate while unobserved, or any personality trait. This is how the PRD's two requirements, "never drifts down on neglect" and "quieter than they were," can both hold.
4. **The tick is a deterministic function** `advance(state, events, t₀ → t₁)` that steps in 60-second sub-steps. Hot aviaries are scheduled every minute. Dormant aviaries are scheduled every 15 minutes and run 15 sub-steps per batch. Both schedules must produce byte-identical results, and CI tests that.
5. **Render pipeline boundary.** The server emits *behavior plans* at the granularity of seconds to minutes: perch targets, activity segments, call bouts, bird-to-bird episodes, and weather. The client owns everything finer than about one second: pose curves, micro-motion, note-level call synthesis, particles, and lighting interpolation. The client never advances canonical state.
6. **Raw personality numbers never leave the simulation zone,** with one exception: the account export (§1.3, D4). A server-side *expression compiler* converts traits and mood into quantized behavior, appearance, and voice parameters. Snapshots carry only those parameters.
7. **Presence is gated on the client and made honest on the server.** The client sends a heartbeat only while all three signals hold. The server credits presence in per-minute buckets, takes the max across devices (a union, not a sum), clamps to physical limits, and applies diminishing daily returns with a daily cap.
8. **Greeting choreography runs on the client** from server-compiled per-bird dispositions plus account-level absence length. A returning tab greets within 0.6–2.0 s with no network round trip. The greeting that actually happened is recorded as an event, which the notebook can use.
9. **Offer reactions are resolved on the server, synchronously and deterministically,** against canonical state. The client plays an "anticipatory beat" (the birds notice the item) while the request is in flight. The resolved reaction is stored inside the event, and the tick applies exactly that reaction.
10. **First frame without a spinner.** The edge streams HTML with the snapshot inlined. The core renderer (≤ 60 KB gz) is fetched via 103 Early Hints. If the snapshot is late, the user sees the "quiet field," which never includes a spinner.
11. **Renderer is Canvas2D with cached layers.** A hard performance gate at week 3 on the reference laptop decides whether we switch to WebGL2. The scene graph is renderer-agnostic, so the switch is contained.
12. **All audio is synthesized in an AudioWorklet.** One `CallSpec` object drives both the synthesizer and the caption generator, so a caption always matches what was played. When WebAudio is unavailable, the aviary goes silent and captions default on. CI fails the build if any audio asset ends up in `dist/`.
13. **Notebook, narration, and captions come from an authored generative grammar, not an LLM.** An LLM here would mean sending per-bird interaction data to a third party or hosting a model we can't constrain. A grammar gives us voice control, sparsity control, and output we can lint.
14. **The privacy boundary is enforced by network and IAM topology.** Telemetry has no network route and no credentials to the simulation database. RUM goes to a cookieless domain and carries no identifiers. No product-analytics SDK and no session-replay tool is installed, ever.
15. **Calibration happens in simulation and with consenting dogfood accounts only.** The privacy rule forbids population-level analysis of how birds are interacted with, so production can never be used to tune drift. Constants must be right before launch, and every later change must first be validated in the simulation harness.

### 0.2 PRD commitment → mechanism → proof

| PRD commitment | Mechanism | Proof (automated unless noted) |
|---|---|---|
| First frame is mid-action; no entry animation | Inline snapshot. `pose(t)` is a pure function of plan and time. No entry-transition code path exists. | Playwright: bird pose in frame 1 ≠ rest pose; lint bans spinner/skeleton components |
| Notice, never announce | The design system has no toast, snackbar, or banner primitive. The greeting engine is the whole welcome surface. | ESLint `no-restricted-imports` for toast libraries; copy linter; UI audit checklist at each gate |
| Charm through specificity | Grammar with named slots (bird, perch, weather, comparison). Dedupe window. | Voice linter over 10k generated samples per build; writer reviews 500 samples before each gate |
| Restraint (cap of 7, one screen, sparse chrome) | DB check constraint and engine cap. Layout solver never scrolls or crops. Top bar has 5 fixed items. | Constraint test; layout property test over 400 viewport sizes |
| Drift is monotonic toward expressive | Non-negative drive × headroom term; DB trigger | Property test over 10⁵ random event streams; trigger test |
| No last-write-wins for personality | Append-only event log; single writer per aviary with fenced leases; grants | Concurrency/chaos suite (§13.3) |
| Personality never shown | Expression compiler; snapshot schema has no trait fields; engine package is never bundled into the client | Schema contract test; bundle-content test |
| Presence is honest | 3-signal client gate; server clamps; per-minute max-union | "Tab open but idle for 48 h" persona yields exactly 0 credit; e2e test with a background window |
| Calls are procedural | AudioWorklet synthesizer; no audio assets | CI fails the build on any `audio/*` MIME type in `dist/` |
| No streaks or visit-frequency surfaces | No per-day visit data leaves the simulation zone. Observer inputs are typed to exclude presence. | Type-level test; linter lexicon; review checklist |
| Synthetic account IDs | UUIDs everywhere; email only in the identity zone; HMAC blind index for lookup | Log/telemetry schema scanner; PII canary test (§11.2) |
| Privacy is an architectural rule | Separate network/IAM for telemetry; RUM carries no IDs | IAM policy tests; RUM payload allowlist |
| Accessibility ships in v1 as a designed surface | Narration, captions, and reduced-motion each have their own design and owner | Manual screen-reader matrix; axe; reduced-motion design sign-off is a launch gate |

---

## 1. Scope

### 1.1 In v1

- **Aviary.** One per account. Two starter birds, cap of 7. Six species. Stable bird identities. Renameable birds.
- **Simulation.** Server-side 60 s tick. Personality drift. Attunement. Mood. Day/night in the account's timezone. Ambient weather. Bird-to-bird social behavior. Call bouts, choruses, alarm contagion. New birds arrive by aviary age.
- **Interactions.** Return-greeting. Presence accounting. Listen-in. Offers (seed, song fragment, still pool). Settle with a 5 s undo. Field notebook, read-only and infinitely scrollable.
- **Rendering.** A single responsive scene with three perch zones, idle micro-motion, flight transitions, subtle parallax, ambient leaves and feathers, weather, and lighting. A top bar that fades. The quiet field. The empty-aviary state with fly-in.
- **Audio.** Procedural calls. Chorus mixing. Listen-in mix. A procedural ambient bed (wind, rain, night texture). Silence-with-captions fallback.
- **Accessibility.** Screen-reader narration in naturalist prose. Call captions. Reduced-motion mode as its own designed rendering. Full keyboard operation. WCAG AA contrast. Forced-colors support for chrome.
- **Accounts.** Magic-link sign-in. Per-device sessions with revocation. Verified email change. JSON export delivered by email link. Soft delete (30 days) then hard delete.
- **Visits.** Per-email invitations, off by default. Read-only ambient view. Revocation. 30-day expiry for unused invites. Visit log. Opt-in visit notification email.
- **Operations.** Aggregate-only telemetry. Synthetic performance checks. Tick-latency alarms. Integrity monitors.

### 1.2 Out of v1, and the guardrail that keeps each one out

| Excluded | Guardrail |
|---|---|
| Native apps | Protocols are designed for browsers only: no push tokens, no offline-first sync engine. |
| Gamification (streaks, achievements, levels, counters, visit calendars, "birds adopted: N") | No per-day visit data is exposed by any API. The notebook observer cannot read presence. Copy-linter lexicon (§10.3). Anti-feature review at every gate. |
| Tamagotchi mechanics | The schema has no hunger, health, or happiness-decay fields. Drift cannot go negative. |
| Social network surfaces | Visits are the only social table family. No profile, follow, or comment tables. Invitations cannot carry free text. |
| Notifications or re-engagement | The mailer accepts only allowlisted templates: `sign_in`, `email_change_verify`, `export_ready`, `visit_invite`, `visit_notice` (opt-in only). Any other template ID is rejected in code. |
| Payments, shared or multi-aviary accounts, custom scenes, discovery, leaderboards | `aviaries.account_id UNIQUE`. No cross-account read path exists in the API. No cross-account aggregates are computed. |
| Recorded audio (including as a fallback) | Build check (§8.9). |
| Trait stats in any view, including internal tools | No endpoint returns traits except the export job. Support tooling shows only integrity pass/fail per account. |
| Importing an export | Not built. An import would be a last-write-wins path for personality. |
| Releasing or rehoming a bird | Not in v1 (§1.3, D17). |

### 1.3 Decisions on points the PRD leaves open or contradicts

| # | Question | Decision | Why |
|---|---|---|---|
| D1 | The top bar lists four icons ("nothing else"), but settle must be triggered from the top bar. | Settle is a **fifth** top-bar item: a small dusk glyph, placed last. | Two later, more specific sections require settle in the top bar. The four-icon list reads as the chrome inventory written before settle was placed. |
| D2 | Where does mute live? Does muting affect drift? | "Sound: on/off" lives in the accessibility settings panel, one click from the top bar. Presence with audio audible adds a small bonus to the vocal-frequency drive (×1.25). Muted presence counts fully for every other trait. | The brief names mute as a behavior birds respond to. Monotonicity means muting can only withhold a bonus, never subtract. |
| D3 | "Never drifts down on neglect" vs. "quieter than they were" after two weeks away | The `attunement` state (decision 3 in §0.1; details in §5.6). | Traits stay monotonic. User-directed eagerness reflects what has recently been observed, which is exactly the PRD's wording. |
| D4 | The export includes "current personality vectors," but the numbers are "never exposed." | The export includes raw trait values under `simulation_state` with one plain sentence explaining them. They are never rendered in the product, the export UI shows no preview, exports are rate-limited to one per 24 h, and there is no import. **Flag for product/legal.** | Portability and data-access obligations cover derived data. A downloaded file is not a product surface, and the rate limit keeps it from becoming a stat-checking loop. |
| D5 | Browser autoplay policy vs. "calls already audible" | Create the AudioContext at boot. If it is suspended, the aviary renders silently, and the first user gesture anywhere resumes audio with a 2 s fade-in. We never show a "click to enable sound" prompt. | A prompt would announce. Returning users with media engagement often get autoplay anyway. |
| D6 | Presence activity window ("a few minutes") | Initial value 5 min, delivered in snapshot config so it can be tuned without a client release. Calibrate within 3–8 min during dogfood. | The PRD says to lean long: watching without moving is the product. |
| D7 | `keypress` is deprecated; touch devices rarely emit `pointermove`. | Activity = `pointermove`, `pointerdown`, or `keydown`. | Same intent in modern APIs. A tap on a phone is a sign of being there. Device motion and scroll do not count. |
| D8 | Absence length across devices | Absence = now − latest presence heartbeat from **any** device on the account. | The laptop five minutes ago means the phone gets a glance, not a re-orientation. |
| D9 | Scope of settle | The settled lighting is local to the device. Its canonical effect is a small mood-quieting nudge. The settle event is sent only after the 5 s undo window, so undo never needs server compensation. Presence pings stop immediately. | Settling on the laptop should not put the phone to evening. |
| D10 | Audio in a hidden tab | Fade out over 2 s and suspend the AudioContext. Resume on visible, then greet. | Nothing to see, battery, and hidden time is never presence. Background listening would be attention the engine refuses to count. |
| D11 | What visitors can see and set | Scene and audio only. No notebook, no offers. They get a local accessibility settings panel (captions, reduced motion, sound, narration text), stored in their own browser. | Accessibility must reach visitors too. The notebook belongs to the host. |
| D12 | Lifetime of a claimed visit | Active until revoked or the host's account is deleted (the PRD sets no limit). The visit log is kept for 12 months rolling. Visitor emails on revoked or expired invites are purged after 90 days. | Honors the PRD and minimizes a non-user's PII. |
| D13 | Personal message in an invitation | None. The invitation email is fixed matter-of-fact text. | Prevents chat-by-invite and spam abuse. |
| D14 | Timezone when devices are in different zones | One canonical account timezone: the timezone of the device with the most recent sustained presence (≥ 10 min). It drives both the simulation and lighting. On a change, lighting cross-fades over 30 s and circadian targets slew over 2 h. | Mood and light must agree. A small, natural "jet lag" beats split-brain time of day. |
| D15 | Can the nocturnal species be a starter? | No. Starters are two diurnal species with registers at least ~an octave apart and distinct silhouettes. The nightjar-like species arrives only as a third or later bird. | The first encounter must be legible in daylight. At night, a two-bird aviary still has sleep shuffles, murmurs, and a procedural night texture. |
| D16 | How a new bird "appears in the user's flow" | A wild **candidate** bird starts visiting the far back perch intermittently. The user can focus it to open an adoption card (naturalist voice) or let it go. There is no toast. The notebook may observe it. | Discovery by noticing. Arrivals depend only on aviary age. |
| D17 | Releasing a bird | Not in v1. Birds leave only when the account is hard-deleted. | The PRD does not specify it, and it would conflict with identity continuity. |
| D18 | Session list details | Device label and sign-in date. **No** "last active." | "Last active" edges toward a visit-frequency surface. The sign-in date is enough to spot an unknown device. |
| D19 | Localization | English only in v1. | The grammar would have to be re-authored per language. |
| D20 | Seasons and sunrise times | A fixed canonical day curve in local time, with no latitude and no seasons. | We never ask for location. |
| D21 | Reduced-motion preference scope | Stored at the account level with three values: follow system (default), on, off. | A vestibular need follows the person across devices. |
| D22 | Empty notebook for new users | An adoption entry is written at adoption ("two birds came to the aviary this morning…"). | The notebook is never empty, and the first entry is an observation. |
| D23 | Offer placement | No placement UI. The item lands at the front, horizontally near the focused or listened-in bird if there is one, otherwise near front-center with jitter. | Keeps offers a gesture, not a tool. Keyboard users get targeted placement for free. |
| D24 | Bird-count dimension in performance telemetry | Allowed only as coarse buckets {2, 3–4, 5–7} on frame-timing histograms, with a k-anonymity floor. **Privacy-reviewed exception.** | Needed to ramp bird counts safely (§14.4). It is operational, not relational. |

---

## 2. Architecture

### 2.1 Shape

```
 Browser                              Edge (CDN + worker)              Primary region (multi-AZ)
┌──────────────────────────┐  HTML  ┌───────────────────────┐        ┌────────────────────────────────────────┐
│ core: scene, choreo,     │◀───────│ edge worker           │◀──────▶│ regional snapshot caches (3 regions)   │
│   presence, outbox       │ +inline│  · verify edge token  │        │                                        │
│ lazy: audio worklet,     │ snap   │  · stream HTML        │        │ api (TS/Fastify)                       │
│   prose, notebook,       │        │  · 103 Early Hints    │        │  auth · snapshot · events · offers     │
│   settings, visits       │─events▶│  · immutable assets   │───────▶│  notebook · settings · visits · export │
└──────────┬───────────────┘        └───────────────────────┘        │                                        │
           │ RUM (cookieless domain, no IDs)                          │ sim-workers (leased shards, fenced)    │
           ▼                                                          │ jobs: mailer · export · deletion ·     │
   telemetry collector ──▶ metrics/traces (Zone C)                    │       integrity                        │
   (no network route to Zone A or B)                                  │ Postgres: identity (A) · sim (B)       │
                                                                      │ Redis: rate limits, presence-last      │
                                                                      │ object store: exports (encrypted)      │
                                                                      └────────────────────────────────────────┘
```

**Zones.**
- **Zone A (identity):** email ciphertext, blind index, magic links, sessions, invitations (visitor email), mailer.
- **Zone B (simulation):** aviaries, birds, personality, simulation state, events, notebook, visit grants, visit log.
- **Zone C (telemetry):** aggregate metrics, traces, scrubbed logs.

Zones A and B are separate Postgres schemas with separate roles, both inside one cluster in v1. Zone C is a separate cloud project and VPC. It has no peering to the database subnet, and the IAM policy denies it any database credentials. That is the PRD's "architectural rule, not policy."

**Deployables.** There are only five, which keeps PII boundary crossings few:
1. `edge`: the CDN worker.
2. `api`: a modular monolith.
3. `sim-worker`.
4. `jobs`: mailer, export, deletion, integrity.
5. `web`: static assets.

### 2.2 Stack (or equivalents)

- **Client:** TypeScript and Vite. The scene is framework-free. Chrome panels use Preact (about 4 KB). Canvas2D, AudioWorklet.
- **Edge:** Cloudflare Workers or Fastly Compute. HTTP/3 with TLS 1.3 0-RTT resumption.
- **Origin:** current Node LTS with Fastify; Postgres 16+ (managed, synchronous standby across AZs, PITR); Redis; S3-compatible object storage; KMS for envelope keys.
- **Email:** a transactional provider with a DPA, and with open and click tracking **disabled**. Click-tracking rewrites links, which breaks single-use magic links and leaks behavior.
- **Observability:** OpenTelemetry to a metrics and trace backend in Zone C. Error tracking is self-hosted Sentry or GlitchTip with scrubbing (§11.2).

### 2.3 Monorepo packages

| Package | Runs on | Contents |
|---|---|---|
| `engine` | sim-worker, API (read-only resolvers) | Deterministic tick: drift, attunement, mood, environment, planner, social, observer, lifecycle. **Never bundled into the client** (bundle test). |
| `expression` | server | Traits + mood + attunement → quantized expression parameters |
| `schema` | everywhere | Snapshot, event, and API types; runtime validators (TypeBox/zod); schema versions |
| `choreo` | client | `pose(t)`, plan interpolation and reconciliation, greeting and settle choreography, ambient continuation |
| `voice` | client | Motif grammar → `CallSpec`; `describe(CallSpec)` → caption |
| `prose` | server (notebook), client (narration) | Naturalist grammar, slot fillers, dedupe, linter |
| `scene` | client | Layout solver, rigs, renderer, lighting, particles, quality governor |
| `audio` | client | AudioWorklet synthesizer, mixer graph, ambient bed |

### 2.4 Regions and resilience

- **Primary region:** Postgres with a synchronous standby in a second AZ, so RPO ≈ 0 for an AZ loss. An async cross-region replica gives RPO ≤ 5 min and RTO ≤ 1 h for a region loss. WAL archiving with 35-day PITR.
- **Snapshot caches:** in three regions (Americas, EMEA, APAC). The edge reads the nearest one.
- **If the region is down:** open clients continue with local ambient continuation (§7.5) and queue events for up to 15 min. On recovery, workers catch up the missed ticks deterministically, with no events for the outage window.

---

## 3. Data model

### 3.1 Zone A: identity

```
accounts(
  id uuid pk,                    -- synthetic; the ONLY account identifier anywhere
  created_at timestamptz,
  status text check (status in ('active','pending_deletion')),
  deletion_requested_at timestamptz null,
  tz_canonical text,             -- IANA; see D14
  settings jsonb                 -- a11y prefs, visit_notice_opt_in, offer_shortcut, etc.
)
account_email(
  account_id uuid pk fk,
  email_ciphertext bytea,        -- AES-256-GCM with the per-account DEK
  email_blind_index bytea unique,-- HMAC-SHA256(pepper_v, normalize(email)); lookup only
  dek_wrapped bytea,             -- KMS-wrapped per-account data key (crypto-shred on hard delete)
  verified_at timestamptz
)
email_change_requests(id uuid pk, account_id, new_email_ciphertext, new_blind_index, token_hash, expires_at, consumed_at)
magic_links(token_hash bytea pk, email_blind_index, account_id uuid null, created_at, expires_at, consumed_at)
sessions(id uuid pk, account_id, token_hash, device_label text, created_at, revoked_at)
invitations(id uuid pk, host_account_id, visitor_email_ciphertext, token_hash, status, created_at, expires_at, claimed_at, revoked_at)
```

- Normalization for the blind index: trim, then lowercase the whole address. Unicode is NFC-normalized. Plus-addresses are **not** collapsed, because the user owns that distinction.
- The pepper is versioned in KMS and rotated by dual lookup during a transition window.
- The blind index is a pseudonymous function of the email. It may never appear outside `account_email` and `magic_links`: no logs, no joins in other services.

### 3.2 Zone B: simulation

```
aviaries(
  id uuid pk, account_id uuid unique, created_at, seed bigint,
  shard int,                           -- hash(id) mod 4096
  writer_epoch bigint,                 -- fencing (§6.2)
  last_tick bigint,                    -- floor(unix_seconds/60) of last applied sub-step
  event_cursor bigint,                 -- last consumed per-aviary event seq
  params_version int,                  -- engine constants version (forward-only)
  sim_state jsonb,                     -- volatile state (§3.3)
  sim_state_schema int
)
aviary_event_counters(aviary_id pk, next_seq bigint)   -- separate row to avoid tick lock contention
birds(
  id uuid pk, aviary_id, species_id text, species_asset_gen int,  -- pinned visual generation
  name text check (char_length(name) between 1 and 24),
  status text check (status in ('active','candidate','declined')),
  adopted_at timestamptz null, adoption_order smallint null,
  voice_params jsonb,                  -- frozen at adoption (§8.3); never re-derived
  look_params jsonb                    -- frozen silhouette/palette endpoints, individual markings
)
bird_personality(
  bird_id uuid pk,
  boldness, warmth, vocal, plumage, curiosity   double precision,   -- [0,1]
  ceil_boldness … ceil_curiosity                double precision,   -- per-bird ceilings in [0.75,0.95]
  seed_boldness … seed_curiosity                double precision,   -- v₀, for the headroom normalizer
  drive_boldness … drive_curiosity              double precision,   -- low-pass filter state, ≥ 0
  updated_tick bigint
)
bird_trait_daily(bird_id, local_date date, boldness, warmth, vocal, plumage, curiosity, pk(bird_id, local_date))
interaction_events(
  aviary_id uuid, seq bigint, event_key uuid,   -- idempotency (UUIDv7 from client)
  type text, payload jsonb, device_session text, -- random per-tab id, not linked to the session token
  client_ts timestamptz, received_at timestamptz,
  pk(aviary_id, seq), unique(aviary_id, event_key)
) partition by range (received_at)              -- daily partitions, dropped after 30 days
notebook_entries(id uuid pk, aviary_id, local_date date, created_tick bigint, observation_type text,
                 template_id text, bird_ids uuid[], body text)
visit_grants(id uuid pk, invitation_id uuid, aviary_id, grant_token_hash, created_at, revoked_at)
visit_log(id uuid pk, invitation_id uuid, aviary_id, started_at, approx_duration_s int)
exports(id uuid pk, account_id, status, object_key, created_at, expires_at)
shard_leases(shard int pk, owner text, epoch bigint, expires_at timestamptz)
```

### 3.3 The `sim_state` blob (volatile, written only by the tick)

```ts
{
  tz: { canonical: string, pending?: { tz: string, since_tick: number } , slew_from_offset_min?: number },
  weather: { kind: 'clear'|'rain'|'wind', intensity: number, start_tick, end_tick, next_rain_tick, next_wind_tick },
  startle: { next_tick },                       // rare ambient startles (twig creak, passing shadow)
  offers: [{ id, kind, pos:{x,zone}, placed_tick, expires_tick, reaction:{…} }],
  presence: { last_at, local_date, day_credit, audible_share },     // account-level, minute-bucketed
  birds: { [bird_id]: {
      affect: { arousal, ease, interest }, mood: MoodLabel, mood_since_tick,
      attunement: number,
      perch: { zone:'front'|'middle'|'back', slot:number }, plan: PlanSegment[],   // horizon (§5.9)
      offer_cooldown_until_tick: number, accepted_offers_today: number,
      listen_credit_today: number
  }},
  social: { affinity: { [pairKey]: number } },  // medium timescale, bird-to-bird
  observer: { tokens: number, memory: ObserverMemory },  // 14-day rolling facts (first greeter per day, etc.)
  lifecycle: { next_candidate_age_days: number, candidate?: { bird_id, since_tick, visits_scheduled: number[] } }
}
```

### 3.4 Invariants enforced in the database

- `bird_personality`: a `BEFORE UPDATE` trigger raises if any trait decreases (tolerance 1e-12), if any drive is negative, or if any trait exceeds its ceiling. `DELETE` is granted only to `deletion_worker`.
- `UPDATE` on `bird_personality`, `aviaries.sim_state`, and `notebook_entries` is granted only to `sim_writer`. The API role (`api_rw`) can only `INSERT` events and update `birds.name` (renames are metadata, so last-write-wins is fine for names).
- `notebook_entries` is append-only: no `UPDATE`/`DELETE` except by `deletion_worker`.
- A trigger rejects inserting an 8th `active` bird for an aviary.
- `birds.id` is never regenerated. Migrations touching `birds` or `bird_personality` need a two-reviewer approval label, enforced in CI.

### 3.5 Retention

| Data | Retention |
|---|---|
| Raw interaction events | 30 days (partition drop). Needed for recovery replay and observer windows. |
| Personality, sim state, trait daily history, notebook | Life of the account. Removed at hard delete. |
| Visit log | 12 months rolling |
| Visitor emails (revoked/expired invites) | 90 days, then purged |
| Magic links / email-change tokens | 24 h after expiry |
| Exports | Object and link expire after 7 days |
| Logs (Zone C) | 14 days. May contain account UUIDs, never email or content. |
| Backups | 35-day PITR. A deletion tombstone list is re-applied on any restore. |

---

## 4. API surface

### 4.1 Conventions

- JSON over HTTPS. Every payload is validated against `schema`.
- Session: an opaque 256-bit token in a `__Host-` cookie (`HttpOnly; Secure; SameSite=Lax`). Mutations also require the `X-Aviary-CSRF` header.
- Edge token: an Ed25519-signed cookie, 24 h, carrying `{aviary_id, session_id, exp}`. It is refreshed on every snapshot pull. The edge checks a KV revocation set for revoked session IDs (propagation ≤ 60 s).
- All IDs in URLs are UUIDs. Emails never appear in URLs, query strings, or logs.
- System errors use a typed code, which the client maps to matter-of-fact copy (§4.6). The server never sends user-facing prose for errors.

### 4.2 Endpoints

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /auth/magic-link` `{email}` | Request a link | Always `202`, so account existence is never revealed. Rate limits: 5 per 15 min per blind index; 30 per hour per /24 IP bucket (Redis, hashed keys). Sets a `pending_signin` cookie in the requesting browser. |
| `GET /auth/verify?t=` | Landing | Same browser as the request → auto-POST. Otherwise → a "Continue signing in" button. Link scanners can't consume the token. |
| `POST /auth/verify` `{t}` | Consume | Atomic `UPDATE … SET consumed_at=now() WHERE token_hash=$1 AND consumed_at IS NULL AND expires_at>now()`. Creates the account plus the adoption bootstrap on first sign-in. `303 /`. |
| `POST /auth/sign-out` | End session | |
| `GET /account/sessions` · `DELETE /account/sessions/:id` | Device list and revocation | Revocation writes to the edge revocation set. |
| `POST /account/email-change` · `POST /account/email-change/confirm` | Verified email change | The old email keeps working until confirmation. |
| `GET/PUT /account/settings` | Accessibility prefs, visit notice, offer shortcut, timezone report | |
| `POST /account/export` | Queue export | 1 per 24 h. The link is emailed. |
| `POST /account/delete` · `POST /account/restore` | Soft delete / "I changed my mind" | |
| `GET /aviary/snapshot?reason=open\|visible\|resume\|keepalive` | Canonical snapshot | `ETag = tick:planVersion`; `304` when unchanged. Also served inline by the edge. |
| `POST /aviary/events` `{events:[…]}` | Batch of interaction events | ≤ 50 per batch; idempotent by `event_key`; `sendBeacon`-compatible. |
| `POST /aviary/offers` `{kind, fragment_id?, near_bird_id?}` | Place an offer | Returns `{offer_id, reaction}` synchronously (§5.11). |
| `GET /aviary/notebook?before=&limit=20` | Notebook page | Cursor pagination, newest first. |
| `PATCH /birds/:id` `{name}` | Rename | Validated (§11.6). |
| `POST /onboarding/starters` `{names:[a,b]}` | Name the starters | Only while the aviary is in the `adopting` phase. |
| `POST /aviary/candidate/:bird_id/adopt` `{name}` · `…/let-go` | New bird | Only while a candidate exists. Enforces the cap. |
| `POST /invitations` `{visitor_email}` · `GET /invitations` · `DELETE /invitations/:id` | Invitations | ≤ 10 per day and ≤ 30 outstanding per host. |
| `GET /visits/log` | Host visit log | |
| `GET /v/:token` → `POST /v/:token/claim` | Visitor landing and claim | The claim sets a `__Host-visit` cookie bound to the grant. |
| `GET /visit/snapshot` | Visitor snapshot | `410` if revoked, expired, or host deleted. |
| `POST /visit/heartbeat` | Visit-duration accounting | Writes only `visit_log`. **Never** the event log. |
| `POST https://rum.<cookieless-domain>/v1` | Aggregate RUM | No cookies, no IDs (§12.3). |

### 4.3 Interaction events

```ts
type Event =
 | { type:'presence', event_key, client_ts, window_id:string, active_s:number /*0..30*/, audible:boolean }
 | { type:'listen_in', event_key, client_ts, bird_id, action:'start'|'end' }
 | { type:'greeting_observed', event_key, client_ts, reason:'open'|'visible'|'resume',
     greeters:{bird_id, form, offset_ms}[], absence_s:number }
 | { type:'settle', event_key, client_ts }        // sent after the 5 s undo window
 | { type:'reengage', event_key, client_ts }
 | { type:'tz_report', event_key, tz:string }
```

**Server-side validation:**
- `active_s` is clamped to the elapsed wall time since that `window_id`'s previous heartbeat plus 5 s, and to ≤ 30.
- Heartbeats older than 15 min on arrival are dropped.
- More than one presence event per 25 s per `window_id` is dropped.
- A `listen_in` for a bird that isn't in the aviary is rejected.
- The event transaction does `UPDATE aviary_event_counters SET next_seq = next_seq + n RETURNING` and then inserts. The counter row lock is held until commit, so **per-aviary seq order equals commit order**. That removes the classic cursor-skip hazard of global identity columns.

### 4.4 Snapshot (host view)

```ts
{
  schema: 4, server_now: number, tick: number, next_tick_at: number, plan_version: string,
  tz: string, light: { phase: 'night'|'dawn'|'day'|'dusk', sun: number /*0..1*/ },
  weather: { kind, intensity, start_at, end_at },
  offers: [{ id, kind, pos, placed_at, expires_at, reaction }],          // includes unconsumed offer events (§6.4)
  birds: [{
    id, name, species, look: { palette: OKLCH[], detail: number /*0..15*/ , markings },   // compiled; no trait values
    voice: VoiceParams,                        // frozen signature + current modulation (quantized)
    mood: MoodLabel,                           // label only; drives narration and captions
    expr: { idle: {preen,scan,rest,tilt,shuffle}, tempo, approach, gaze_at_viewer, call_rate, respond },  // quantized to 1/16
    greet: { weight, latency_bias, style, repertoire:number[] },
    plan: PlanSegment[]                        // [{t0,t1,kind,args}] kinds: perch|fly|preen|scan|rest|tilt|
                                               //   forage|drink|bathe|call_bout|respond|chorus|sleep|startle
  }],
  candidate?: { bird_id, species, look, voice, plan },
  last_presence_at: number | null,             // host only
  config: { presence_window_s: 300, heartbeat_s: 30, keepalive_s: 60 }
}
```

Size: 7 birds with a 180 s plan is about 6–9 KB of JSON, about 2–3 KB gzipped. The **visitor snapshot** is the same, minus `last_presence_at`, `candidate` adoption controls, and `config.presence_*`. Bird IDs are replaced with per-snapshot handles `b1…b7`.

### 4.5 Visit flow

1. The host submits the visitor's email. The API (Zone A) creates the `invitation`: visitor email encrypted with the host's DEK, a token hash, `expires_at = now + 30 d`. The mailer sends the fixed `visit_invite` template, which contains the link `/v/<token>`, the host's chosen display text ("A Pocket Aviary"), and no host email.
2. The visitor opens the link. The landing page (matter-of-fact) has a "View aviary" button, so scanners don't claim the invite. `POST claim` atomically sets `claimed_at` if the invite is still pending and unexpired. It then creates a `visit_grant` and sets the `__Host-visit` cookie. A second claim attempt gets the "no longer available" surface.
3. The visitor client loads `/visit` (edge-rendered like the host view) and pulls `/visit/snapshot` every 60 s. The grant is validated at origin on every pull, with a 10 s cache.
4. Visitor heartbeats (every 60 s) update one `visit_log` row: `started_at`, and `approx_duration_s` rounded to 5 min. Nothing reaches `interaction_events`. The visit token has no write scope for any aviary endpoint; the gateway enforces scope and a contract test proves it.
5. If the host opted in, `visit_notice` is emailed at most once per invitation per day: "Someone you invited (<visitor email>) visited your aviary on <date>."
6. Revocation sets `revoked_at` on the invitation and grant. The visitor's next pull (≤ 60 s + jitter) returns `410`, and the client replaces the scene with the matter-of-fact surface. Unclaimed revoked, expired, or deleted links show the same surface.
7. While the host is pending deletion, all grants return `410`. At hard delete, invitations, grants, and visit logs are deleted.

### 4.6 Error codes → system copy (matter-of-fact register)

| Code | Copy |
|---|---|
| `auth.link_invalid` (expired, used, or unknown) | We couldn't sign you in. The link may have expired. Try requesting a new link. |
| `auth.session_expired` | Your session timed out. Sign in again to keep watching. |
| `aviary.load_failed` (no snapshot after ~8 s of retries) | Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch. |
| `visit.unavailable` | This visit is no longer available. |
| `browser.unsupported` | Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge. |
| `export.queued` | We're preparing your export. We'll email a download link to your address. |
| `delete.pending` | Your account is scheduled for deletion on {date}. [I changed my mind] |

System messages appear in a persistent inline panel in the top-bar region or in the settings view. They are never toasts, and they are never dismissed on a timer.

---

## 5. Simulation engine

### 5.1 Clock and scheduling

- Simulation time is quantized to `tick = floor(unix_seconds / 60)`. `advance(state, events, from_tick, to_tick)` applies one sub-step per tick index.
- **Hot schedule** (a snapshot pull, a presence heartbeat, or an event in the last 30 min): advance each minute at `:00 + jitter(0–5 s)` per shard. The planner horizon is 180 s.
- **Dormant schedule:** advance every 15 min (15 sub-steps). The planner horizon is 20 min, so any published snapshot is valid until the next batch.
- **Promotion:** an event or snapshot request on a dormant aviary sends a promote message to the owning worker. The worker catches up to `now_tick` (at most 15 sub-steps, well under a millisecond of CPU) and publishes before its next hot tick. The snapshot endpoint does not wait: the published dormant plan already covers "now."
- **Equivalence property:** for any event history, the two schedules give byte-identical `sim_state` and personality at each 15-minute boundary. Event bucketing is by `received_at` tick in both. Enforced in CI (T10).

This makes the tick run "whether or not any client is connected" in simulated time, at a fleet cost that scales with activity. It is not recomputation from history: each run advances the persisted canonical state from its last checkpoint.

### 5.2 Determinism

- RNG: a counter-based generator (Philox4x32 or SplitMix64 streams) keyed by `(aviary.seed, tick, stream_id)`, with a stream per subsystem so adding a subsystem doesn't shift the others.
- Math: only IEEE-exact operations in the engine (`+ − × ÷ sqrt`). `exp`, `sin`, and `pow` come from a vendored, pure-TS polynomial implementation, so results don't depend on the engine or version. Reductions use a fixed iteration order.
- Golden fixtures: 50 recorded aviary histories are replayed on every CI run. Any byte change needs a `params_version` bump and a reviewed diff report.

### 5.3 Per-sub-step pipeline

```
advance_one(state, tick, events_in_tick):
  env      = environment(state, tick)             // local phase from canonical tz; weather step; startle step
  inputs   = aggregate(events_in_tick)            // §5.4
  for bird in active birds (stable order by adoption_order):
     attunement_step(bird, inputs)                // §5.6
     drive_step(bird, inputs)                     // §5.5 low-pass
     drift_step(bird)                             // §5.5 monotone
     affect_step(bird, env, inputs, social)       // §5.7 OU toward targets + impulses
     mood_classify(bird)                          // hysteresis + dwell
  social_step(state, env)                         // contagion, affinity, response and chorus scheduling
  planner_step(state, env, horizon)               // §5.9 extend plans
  offers_step(state, tick, inputs)                // apply recorded reactions; cooldowns; expiry
  observer_step(state, env, inputs)               // §5.12
  lifecycle_step(state, tick)                     // §5.13 candidate visits and arrival ages
  daily_rollover_if_local_midnight(state)         // reset day counters; write bird_trait_daily
```

Ticks read nothing outside `state`, `events_in_tick`, `params(params_version)`, and the static species tables.

### 5.4 Input aggregation (presence honesty on the server)

- **Per-minute presence credit** `p ∈ [0,1]`: sum each device window's clamped `active_s` inside the minute, then take the **max across devices**. That approximates the union of attention without letting two open devices double-count. The small undercount when devices alternate within a minute is acceptable.
- **Diminishing daily returns:** `effective = p · g(P_day)`, where `P_day` is the effective minutes already credited today (local date). `g = 1` for the first 20 min, then `g = 1 / (1 + (P−20)/20)`. The daily cap is 40 effective minutes (2.0 units, where 1.0 unit = 20 effective minutes).
- **Audible share:** the fraction of credited presence seconds with `audible=true`.
- **Listen-in credit per bird:** the listen-in interval intersected with presence seconds in the same minute. 1.0 unit = 5 min per day, capped at 2.0 units per day per bird.
- **Offers:** "near" credit goes to every bird within a radius of the placement (boldness). "Accepted" credit goes to the reacting bird (curiosity), counted only if the per-bird cooldown had elapsed, and capped at 3 accepted offers per bird per day.
- **Settle and reengage** feed mood only, never drift.
- Visitors produce no inputs. Their endpoints cannot write here.

### 5.5 Drift function

Per bird `b` and trait `k`, per sub-step (`Δt = 1/1440` day):

```
u_bk   = Σ_s w_ks · signal_s(b)                      // daily-rate units, ≥ 0
D_bk  ← D_bk + (Δt / τ_D) · (u_bk − D_bk)            // low-pass; τ_D = 3 days; D ≥ 0 always
Δv_bk  = r_k · D_bk · (c_bk − v_bk)² / (c_bk − v0_bk) · Δt
v_bk  ← min(c_bk − ε, v_bk + max(0, Δv_bk))
```

- **Monotonic:** `D ≥ 0` and `c > v`, so `Δv ≥ 0`. The `max(0, ·)` is defensive, and the DB trigger backs it up.
- **Keeps drifting just after the user leaves, from pre-departure inputs:** `D` decays with τ = 3 days after the last presence, so a small, tapering drift continues. That is exactly "personality drifts during the user's absence based on inputs from before they left." Nothing is invented at return.
- **The squared headroom term** starts at the linear rate and slows hyperbolically as a trait nears its ceiling. Heavy long-term users keep noticing small change for years instead of saturating in months.
- **Per-bird ceilings** `c ∈ [0.75, 0.95]`, drawn at adoption, keep birds distinct even after years of attention.

**Initial weights** (the calibration harness refits them; units as defined in §5.4):

| Trait | Presence | Listen-in (this bird) | Offer near | Offer accepted | Song fragment responded | `r_k` initial |
|---|---|---|---|---|---|---|
| boldness | 1.0 | 0.3 | 0.5 / event | — | — | 0.011 |
| warmth | 0.6 | 1.0 | — | — | 0.2 | 0.011 |
| vocal frequency | 0.4 (×1.25 if audible) | 1.0 | — | — | 0.3 | 0.010 |
| plumage saturation | 1.0 | 0.3 | — | — | — | 0.011 |
| curiosity | 0.5 | 0.2 | — | 1.0 / event | 0.4 | 0.010 |

**Worked check (plumage, regular persona).** The persona is 5 visits per week, 12 presence-minutes per visit, so the average drive is about 0.43. With `v₀ = 0.35`, `c = 0.85`, `r = 0.011`:
- Day 7 (filter still ramping): Δ ≈ 0.012, which exceeds the instrument threshold of 0.010.
- Day 21: Δ ≈ 0.045–0.05, which is the JND.
- The heaviest single day possible (2.0-unit cap) adds ≤ 0.011 over the following week, about 22% of the JND. A single session is never visible.

**JNDs are defined behaviorally** and measured by the harness through `expression` and `choreo` statistics. The compiler's slopes are set so that Δtrait ≈ 0.05 ≈ 1 JND:

| Trait | JND |
|---|---|
| boldness | +10 percentage points of front-zone dwell share |
| warmth | +10 pp greet-first probability in a 2-bird aviary |
| vocal | +15% call-bout rate when unobserved |
| plumage | mean ΔE2000 ≥ 2.5 across plumage regions |
| curiosity | +10 pp offer-approach probability, +15% head-tilt rate |

**Calibration targets.** Each is a CI assertion in `engine/calibration`:

| ID | Target |
|---|---|
| T1 | Regular persona: every trait with presence weight ≥ 0.5 shows Δ ≥ 0.010 by day 7. |
| T2 | Regular persona: boldness and plumage cross 1 JND at day 21 ± 4. No trait crosses by day 10. |
| T3 | Any persona: no single day contributes more than 0.25 JND to any trait. |
| T4 | "Open but idle for 48 h" (visible, focused, no activity) and "background window for 48 h": Δ = 0 exactly. |
| T5 | All personas, all sub-steps: Δ ≥ 0. |
| T6 | Regular for 3 weeks, then 14 days away: traits ≥ their values at departure. Attunement at return ∈ [floor, 0.6]. A greeting still occurs (≥ 1 greeter). |
| T7 | Heavy daily persona over 2 years: every trait ≤ c − 0.02. Median headroom used at 1 year ≤ 70%. |
| T8 | Listen-in on one bird only for 3 weeks: that bird's warmth Δ ≥ 1.5× the other bird's. |
| T9 | Offer every 10 s for 3 h: curiosity Δ ≤ the Δ from 3 accepted offers. |
| T10 | Hot schedule vs. dormant batching: byte-identical state. |

**Personas:**
- regular
- daily-long (60 min per day)
- weekend-only
- sporadic (one visit every 10 days)
- two-week absence
- idle-tab
- background-window
- listen-in-only
- offer-spammer
- screen-reader (keyboard activity every 4–6 min; see §9.6)
- multi-device overlap (laptop and phone present in the same minutes)

### 5.6 Attunement

- `attunement ∈ [0.35, 1]`. Per sub-step it moves toward 1 in proportion to effective presence credit (about 6 h of presence spread over a week brings it from the floor to about 0.9). Without presence it decays toward the floor with a 4-day half-life.
- **It affects only these parameters:**
  - `greet.weight` for non-primary greeters
  - `expr.approach` toward the viewer's front slots **during presence**
  - `expr.gaze_at_viewer`
- **It never affects:** the ease or wary axis, plumage, the unobserved call rate, perch choice outside presence, or any trait. Enforced by a unit test that sweeps attunement and asserts those outputs are invariant.
- After two weeks away: one bird still notices within 2 s, because the primary greeter is unconditional. Secondary greeters are fewer. The primary's greeting is *more* elaborate, because absence length raises intensity. The birds are quieter toward the user, with no wariness. That matches the PRD's return picture.

### 5.7 Mood

**Affect model.** Each bird has continuous affect: `arousal`, `ease` (wary ↔ settled-in), and `interest`, all in [0, 1]. Each evolves as an Ornstein–Uhlenbeck process toward a target:

```
x ← x + θ_x (target_x − x) Δt + σ_x √Δt · N(0,1) + impulses
θ: arousal 1/20 min, ease 1/30 min, interest 1/10 min;   σ small (≈0.02/√min), seeded
```

**Targets:**
- `arousal*`: a species circadian curve in canonical local time (diurnal: 0.05 at night, a 0.85 peak from 06:30–08:30, 0.55 at midday, 0.25 by 20:00; the nightjar-like species is inverted with a dusk and late-evening peak) × weather (rain 0.9, wind 1.1) + 0.1·interest.
- `ease*`: 0.45 + 0.35·boldness + 0.1·warmth(social proximity) − alarm contagion − wind·(1 − boldness)·0.2 + recent positive impulses.
- `interest*`: 0.3 + 0.5·curiosity·novelty, where novelty comes from offers, song fragments, new birds, and falling feathers the bird can see.

**Impulses** (added to `x`, each decaying through the OU process):

| Event | Effect |
|---|---|
| Offer accepted | ease +0.15, interest +0.2 |
| Offer placed near a wary bird | interest +0.1 (it watches, then comes) |
| Listen-in (focused bird) | ease +0.05 per minute, capped |
| Settle | arousal −0.1 for all birds (the "small mood-quieting signal") |
| Alarm call from a neighbor | ease −0.25 × proximity × (1 − boldness), recovering within 5–15 min |

**Discrete mood** is a classification with hysteresis (±0.05) and a minimum dwell of 4 min. Salient impulses (alarm, accepted offer) can bypass the dwell.

| Mood | Condition |
|---|---|
| `roosting` | arousal < 0.12 (narrated as "settled" or "asleep") |
| `drowsy` | arousal < 0.30 |
| `wary` | ease < 0.35 |
| `curious` | interest > 0.60 |
| `alert` | arousal > 0.70 |
| `content` | otherwise |

- **Daily-ish reset:** the dawn rise in the circadian target pulls arousal and ease back toward each bird's personality baseline. That is the reset, and it happens in simulated time, usually while the user is away. The tab opening never resets mood.
- **Persistence across sessions** needs no special handling: mood is canonical state that keeps evolving.

### 5.8 Environment

- **Day/night:** `sun(t_local)` is a smooth fixed curve: dawn 05:30–07:30, dusk 18:00–20:30, night 21:00–05:00. It drives lighting on the client and circadian targets in the engine.
  - DST and timezone changes: lighting re-anchors with a 30 s cross-fade; circadian targets slew over 2 h.
- **Weather:** deterministic per aviary from the seed.
  - Rain: Poisson, λ = 3 per week, 6–18 min, biased toward dawn and afternoon.
  - Wind: 4 gusts per day, 1–4 min each.
  - No storms, no snow.
  - Rain multiplies the call-bout rate by 0.4 during the rain and by 0.7 for 10 min after. This is an **expression-level** effect, never a trait change.
  - Wind raises arousal and triggers a wary impulse in low-boldness birds.
- **Startles:** 1–3 per day. A visible, subtle cause (a creaking branch, a passing cloud shadow) produces an alarm call from the most reactive bird, and the contagion spreads. Startles are never tied to presence or absence.

### 5.9 Behavior planner

The planner extends each bird's plan to the horizon. Segments carry server-time bounds and are immutable once published, except through reconciliation (§6.5).

- **Perching:**
  - Hazard of moving: once per 6–15 min by species, × (0.5 + arousal), × (1 + interest).
  - Zone choice: `softmax(U_zone / 0.3)`, where `U_front = 1.2·expr.approach + 0.8·ease + [presence_now]·0.6·attunement − crowding`, `U_back = 1 − ease + 0.3·[raining]`, middle is the average, and each gets `+ affinity` to neighbors in that zone.
  - Slots come from the layout's slot table (§7.3). Slot assignment is canonical, so every device shows the same bird on the same slot.
- **Activities:** a semi-Markov process over {rest, preen, scan, tilt, shuffle, forage (seed present), drink/bathe (pool present), sleep}.
  - Durations come from species × mood distributions: content preens more; wary scans and sits back; curious tilts; drowsy fluffs low.
  - Presence-now raises `gaze_at_viewer` segments for warm, attuned birds. That is how the birds register idle attention at minute scale; the client adds second-scale glances (§7.4).
- **Call bouts:**
  - Hazard = species base × `expr.call_rate` × mood factor (alert 1.3, content 1.0, curious 1.1, wary 0.6 but with alarm-type motifs, drowsy 0.4, roosting 0.05 as murmurs; the nightjar is active at night) × time of day (dawn-chorus factor 2.0 from 05:45–07:30) × weather.
- **Responses:** when bird A's bout starts, each other bird B responds with P = `warmth_expr(B) · affinity(A,B) · moodFactor(B)`, after 0.5–3 s. That produces call-and-response.
- **Choruses:** when ≥ 2 birds with `call_rate` above threshold have bouts overlapping within 20 s, the planner may promote them to a `chorus` episode with a leader and joiners. Dawn makes this far more likely.
- **Affinity** grows through co-perching and responses and relaxes slowly toward a baseline. It shapes proximity and response probability. It is not a personality trait.

### 5.10 Greeting dispositions (compiled server-side, choreographed client-side)

Per bird: `greet.weight = f(boldness, warmth, ease, arousal, front-proximity)`, `latency_bias`, `style` (a stable per-bird style chosen at adoption from the species repertoire, so each bird greets in its own way), and `repertoire` (allowed forms, ordered by intensity: glance, head-tilt, two-note call, step-forward, hop closer, hop closer plus a longer call).

The client algorithm is in §7.5. The server only compiles the inputs and later consumes `greeting_observed`.

### 5.11 Offer resolver

The API runs `resolve_offer(state_snapshot, pending_offers, offer, rng(seed, offer_id))` synchronously under `pg_advisory_xact_lock(aviary)`. It is pure `engine` code, read-only.

**Rules:**
- Only one offer is in the scene at a time.
- A new offer within 20 s of the previous one returns the existing offer with no new event.
- After 20 s, a new offer replaces the previous item.
- Birds in cooldown (4 min) or over the daily cap still *notice* the item (glance, tilt), but they neither approach nor accrue drift.

| Offer | Curious or content, high curiosity | Wary | Drowsy or roosting | Other shaping |
|---|---|---|---|---|
| seed | Approaches within 3–10 s, takes it, returns | Watches 20–90 s, then approaches slowly, may hesitate | Glances; usually stays | Boldness shortens approach latency |
| song fragment | Joins in (the response motif borrows the fragment's tempo and tonal center) | Goes quiet and watches | Silent | Vocal frequency raises P(join); low warmth raises P(call against) |
| still pool | Drinks; if bold and content, bathes | Watches, drinks later | Watches | Pool lasts 10 min, evaporating as a slow fade |

The response `{offer_id, reaction: [{bird, segments[] }]}` is returned to the client and stored in the event payload. When the tick consumes the event, it applies the **recorded** reaction to the plan and credits drift from its outcomes. Result: the device saw exactly what became canonical.

The song-fragment library has 8 fragments, stored as note data and played through a soft, non-bird "offering voice" in the synthesizer. It is never a recording.

### 5.12 Notebook observer

- **Detectors** run each sub-step on typed inputs. The input type deliberately **excludes presence and absence data**. Detectors emit candidate observations with a salience score:
  - Greet order ("pip greeted before wren today, first time this week"; allowed: first time, again, for a second morning; **forbidden:** counts that imply visit frequency, such as "every day this week" or "five days running")
  - Perch change over time, which is how drift becomes observable without numbers ("wren has been taking the front rail more often lately")
  - Weather reactions
  - First bath in a pool
  - Chorus episodes (first, and a notable dawn chorus)
  - Call-and-response pairs
  - A long preen or quiet stretch
  - Candidate bird visits and arrivals
  - The nightjar's night calls
  - Song-fragment responses
  - Alarm and recovery
  - Plumage "looking fuller in the morning light" (fires once per JND crossing, phrased comparatively, with no numbers)
- **Sparsity budget:** a token bucket that refills 0.4 per day with capacity 2. An observation is emitted only if salience ≥ `threshold(tokens)`. At most 1 entry per local day. Arrival and adoption entries may overdraft to −1.
- **Timing:** the observer holds candidates until a natural writing moment (mid-morning or evening in local time), then writes the best one. Entries are dated in local time ("tuesday —").
- **Prose:** `prose.notebook(observation, slots, rng)`. The template ID and a hash of the slot values go into a 30-entry dedupe window.
- **Never written:** anything that references the user, absence duration, visit counts, or trait names or numbers. These are enforced both by the input types and by the linter.

### 5.13 Bird lifecycle

- **Adoption bootstrap (first sign-in):**
  - Pick two diurnal species with register separation ≥ 10 semitones and distinct silhouettes.
  - Seed traits from `U(0.2, 0.45)` plus species bias. Ceilings from `U(0.75, 0.95)`.
  - Freeze `voice_params` and `look_params`: individual pitch offset, signature motif, timbre, markings.
  - Offer name suggestions from a curated list (avoiding species words).
- **Arrival schedule by aviary age:** candidate appearances at about 85, 150, 220, 300, and 420 days (± 10 d per aviary). The schedule is config, keyed only to `aviaries.created_at`.
  - A candidate visits the back perch in 2–4 short bouts per day for up to 14 days, then leaves for 30 days and returns (same or a different species).
  - "Let it go" means another species comes at the next interval.
  - The candidate's voice is allocated to a free register band, with ≥ 2 semitones separation from any same-species bird (§8.3).
- **Cap:** enforced at 7 by trigger and engine. At 7 birds, candidates stop.
- **Identity:** a bird's UUID, `voice_params`, `look_params`, and `species_asset_gen` are permanent. A species art or voice update creates a new generation for **new** birds only, unless it passes the golden-call and golden-render similarity checks (§13.6).

### 5.14 Expression compiler

`compile(bird, traits, mood, affect, attunement, env) → {look, voice-modulation, expr, greet}`

- Outputs are quantized (1/16) and mixed, so no single output is an affine readout of a trait. This is not cryptographic secrecy. It ensures there is no stats surface and no trivial extension that reads traits.
- **Plumage:** interpolated in OKLCH between the species' muted and rich palette endpoints, with chroma capped at about 0.12 (the calm palette). `detail` sets feather-detail layers.
- **Mapping changes:** a change to the compiler's mapping ships with a per-aviary 7-day blend (old → new), so no bird visibly "snaps" after a deploy.

---

## 6. Sync model

### 6.1 One canonical record

Every device and every visitor reads the same published snapshot for the aviary. No client holds state that another client needs. The only client-local state is presentation: settled lighting, listen-in mix, caption and narration queues, and the frozen keyboard focus order.

### 6.2 A single writer per aviary

- Aviaries hash to 4096 virtual shards.
- Workers lease shards through `shard_leases`: TTL 30 s, renewed every 10 s. Every acquisition increments `epoch`.
- Every tick commit is `UPDATE aviaries SET … , writer_epoch=$e WHERE id=$a AND writer_epoch <= $e AND last_tick=$expected`. A paused or zombie worker with a stale epoch fails its write and drops the shard. Split-brain cannot commit.
- `bird_personality` updates run in the same transaction as the `aviaries` update, the `event_cursor` advance, and any new notebook rows. State, cursor, and side effects commit atomically, so events are consumed **exactly once**.

### 6.3 Event ordering and late arrival

- Events are consumed in per-aviary `seq` order, which is commit order (§4.3). Each is bucketed into the tick of its `received_at`.
- An event that arrives after its natural tick was processed is applied in the next tick. Drift contributions are additive and commutative within a day, so lateness changes no outcome, only timing.
- There is **no retroactive recomputation**, ever.

### 6.4 Snapshot propagation

1. After commit, the worker publishes the compiled snapshot to all three regional caches (`snap:{aviary_id}`), with `last_presence_at` merged in from Redis.
2. Offers not yet consumed are overlaid at read time from a small `pending_offers:{aviary_id}` Redis list, written by the offer endpoint. A second device sees an offer on its next pull, not a minute later.
3. The client pulls at `next_tick_at + U(2, 8) s`, which spreads load. It also pulls on `visibilitychange → visible`, on a frame gap > 5 s, and on `online`. `304` responses make this cheap.
4. Visitors use the same path through the visit endpoint.

### 6.5 Client reconciliation

- **Clock:** an offset from `server_now` and the RTT (median of the last 5), smoothed. Plans are rendered in server time.
- **When a new plan arrives:** segments already under way keep playing if the new plan agrees at `now`. Otherwise the bird switches at the next natural boundary (≤ 3 s) or eases into the new target along a short flight or shuffle. **No teleports while visible.** After a hidden period or suspend, the current state is rendered directly, because nobody was watching.
- **Plan starvation** (offline, region outage): `choreo.ambientContinuation` keeps each bird on its perch doing mood-appropriate idle, with seeded variety, indefinitely. There is no freeze and no error, unless a user action needs the server.
- **Outbox:** events are held in memory and mirrored to `sessionStorage` (≤ 200 events, ≤ 15 min old). They are flushed on reconnect, and flushed with `sendBeacon` on `pagehide` or when the page becomes hidden. Idempotency keys make retries safe.

### 6.6 Conflict catalog

| Scenario | Outcome |
|---|---|
| Laptop and phone both present in the same minutes | Presence per minute = max across devices. Listen-in credit is the union per bird. No double-counting and nothing lost. |
| A phone session began before a laptop session ended and writes events afterward | Both event streams are appended. The tick applies additive deltas. Neither device's drift is lost. |
| Rename on two devices | Last write wins on `birds.name` (metadata only). Each device picks up the new name on its next snapshot. |
| Offers from two devices within 20 s | The advisory lock serializes them. The second receives the first offer's reaction. |
| Settle on the laptop while the phone is present | Only the laptop's lighting changes and only its presence ends. A small mood nudge is canonical. The phone keeps watching. |
| Session revoked while the device is open | The next API call returns `401` and the edge denies after ≤ 60 s. The client shows `auth.session_expired`. Queued events for revoked sessions are discarded. |
| A magic link used twice (replay) | The second use gets `auth.link_invalid`. |
| Session times out mid-write | The outbox holds events. After re-sign-in the outbox is flushed only if the account matches. |
| Worker crash mid-tick | The transaction rolls back. The next lease holder catches up from `last_tick`. |
| Region outage | Clients continue locally. Workers catch up after failover. Events older than 15 min are dropped by the client. |
| Bad engine deploy | `params_version` pins behavior. Roll back the code. If data was affected, restore `bird_personality` from `bird_trait_daily` plus PITR, going forward only, and the monotonic trigger blocks any "restore" that would lower traits (it needs an audited override). |

---

## 7. Frontend rendering pipeline

### 7.1 Boot and first frame (returning user, mid-tier phone, 4G profile at 70 ms RTT and 12/4 Mbps)

| t (ms) | Event |
|---|---|
| 0 | Navigation. DNS is cached, and the HTTP/3 0-RTT resumption goes to the edge. |
| ~80 | The edge returns 103 Early Hints (`core.js`, `rig-atlas.webp` ≤ 25 KB) and starts streaming the `<head>`: inline critical CSS, an inline 1 KB script that paints the quiet-field sky from the device clock, and `modulepreload`. |
| 80–180 | The edge verifies the edge token and reads `snap:{aviary}` from the nearest regional cache (p95 ≤ 60 ms), then streams `<script type="application/json" id="snap">…</script>` and `</body>`. |
| ~200 | `core.js` (≤ 60 KB gz; immutable, usually served from HTTP cache with a V8 code cache) executes. |
| ~260 | The layout solver runs, the rigs are built, and the first frame is drawn with birds at `pose(now)` from their plans, mid-preen, mid-scan, mid-call. |
| 260+ | Idle-time lazy loads: `prose` (narration), `audio` plus the worklet, `voice`, and ornaments. Then notebook and settings chunks on demand. |
| 600–2000 | The greeting, if the absence ≥ 30 s. |

**Slow path:** the quiet field (sky gradient for local time, one or two slow motes of light or a drifting leaf) holds until the snapshot arrives. Birds then fade in over 600 ms *already in motion*. That reads as the aviary resolving, not as an entry sequence. After ~8 s of failed retries, `aviary.load_failed` appears. **Spinners, skeletons, and progress bars are banned codebase-wide by lint.**

**Empty aviary (post-adoption only):** the quiet field, then the first starter flies in to its starting slot, then the second 3–6 s later. This path is reachable only once per account (`sim_state.phase = 'adopting' → 'live'`).

### 7.2 Scene composition

The scene has five layers, drawn back to front:
1. **Sky:** a gradient cached per lighting step.
2. **Far foliage:** a cached offscreen canvas.
3. **Perch plane:** branches and rails (cached) plus birds (drawn every frame) plus offer items.
4. **Foreground ornaments:** an occasional near branch, falling leaves and feathers, rain streaks.
5. **Grade:** a lighting and weather color overlay (multiply or soft-light, cached per step).

**Parallax:** layers 2 and 4 offset by ≤ 1% of width, following the pointer (eased, 2 s time constant) plus a slow ambient drift. There is no parallax on touch-only devices and none in reduced-motion mode.

**Backing store:** DPR capped at 2 and at 2560×1440 backing pixels, then upscaled by CSS. Cached layers are re-rasterized only when the lighting step changes (≤ 1 per 10 s) or on a layout change.

**Quality governor:** if p95 frame time exceeds 18 ms over 3 s, degrade in this order:
1. particle count
2. foreground parallax
3. DPR down to 1.5, then 1
4. feather-detail layers

The governor recovers after 30 s of headroom. It **never** lowers bird motion frequency, the motion of the aliveness surface itself.

### 7.3 Layout solver (responsive, no crop, no scroll)

- **Logical scene:** height is 1.0 unit; width is the viewport aspect clamped to [0.46 (9:19.5), 2.33 (21:9)]. Wider viewports get decorative foliage "wings" at the sides, with no perches.
- **Perch slots by zone:** front 3, middle 4, back 4. Slot x-positions are distributed over the usable width with minimum spacing `s_min = bird_width × 1.4`. On narrow portrait viewports the zones stack more vertically: front low, back high, with stronger scale separation.
- **Bird scale by zone** (front 1.0, middle 0.8, back 0.62) × base size = `min(height · 0.14, width / (1.4 · max_slots_per_zone_used))`.
- **Guarantee:** every bird's bounding box plus the flight arcs between any two slots stays inside the safe area (excluding the 44 px top-bar sky band and device safe-area insets). Checked by a property test over 400 viewport sizes × 7 birds × all slot pairs.
- **Resize and orientation change:** birds keep their zone and slot index and ease to the new coordinates over 800 ms (cross-fade in reduced-motion mode).

### 7.4 Bird rig and procedural idle

- **Rig:** body, head, beak (open/close), eye (lid), tail, two wings (folded, half, spread), and legs (perched only). Parts are `Path2D` shapes cached per species and generation. Plumage is drawn with palette fills plus 0–15 detail layers (wing bars, sheen, breast streaks) and a small texture atlas.
- **Motion layers**, summed per frame from seeded noise and never looped:
  - breathing: 0.25–0.4 Hz body scale ±1%
  - head saccades: a quick 120 ms turn, then a hold of 0.4–3 s, mimicking real birds
  - weight shuffles
  - tail flicks
  - feather fluff (drowsy/roosting)
  - blink
  - beak synced to the synthesizer during calls
  - activity clips (preen, scan, tilt, forage, drink, bathe, sleep) as parametric curves with randomized amplitude, timing, and side
- **Frequency ceiling:** no oscillation above 3 Hz except saccades and wingbeats in flight. That avoids strobing.
- **Second-scale "noticing":** while local presence holds, warm or attuned birds (`expr.gaze_at_viewer`) occasionally turn toward the viewer (1 glance per 30–120 s). A focused (listened-in) bird turns a little toward the viewer. Its visuals do not change otherwise; there is no ring, no badge, and no glow.
- **Flight:** a cubic Bézier arc over 0.8–2 s with a wingbeat cycle, ease-in-out. Arcs are constrained to the safe area.
- **Allocation discipline:** the per-frame path allocates nothing. Particles, plan segments, caption nodes, and tween objects come from fixed pools.

### 7.5 Choreography: greetings, settle, offers, plan playback

**Greeting trigger:** a transition into "present-candidate" (visible and focused) when `absence = now − max(last_presence_at, local_last_present_at) ≥ 30 s`, and no greeting has happened in the last 90 s.

```
intensity = clamp(log(absence/30s) / log(2d/30s), 0, 1)
awake = birds not roosting;  if none: pick most wakeful → "sleepy eye-open + soft murmur" form
primary = weighted draw over awake by greet.weight with temperature 0.35   // bolder usually first, not always
schedule primary at U(0.6, 2.0) s, form = pick from primary.style repertoire by intensity (+ seeded variation
          of timing, amplitude, head angle, call motif, number of notes)
for each other bird: P = greet.weight_i · (0.3 + 0.7·intensity) · attunement_i
          schedule at cumulative offset U(1.2, 5.0) s — never in unison
if intensity > 0.8: primary hops toward front and calls longer; one other bird answers (call-and-response)
emit greeting_observed; narration gets a priority entry
```

The form library comes from each bird's style, and every parameter varies per occurrence. A test renders 1,000 greetings per bird and asserts that no two parameter vectors are identical and that the forms are distributed by intensity.

**Settle** (top bar):
- Lighting moves toward the evening grade over 4 s. The master bus drops to −9 dB over 4 s. The warmest awake bird gives one soft call or looks toward the viewer, and the others fluff.
- Presence heartbeats stop immediately. The `settle` event is sent at t + 5 s.
- Undo: any pointer tap or click in the scene within 5 s, or Enter, Space, or Escape for keyboard users, reverses the change over 1 s. No event is sent.
- After 5 s, re-engaging (scene tap or click, a key in the scene, or top-bar interaction) returns the lighting to the current time of day over 4 s and sends `reengage`. A new presence window opens once all three signals hold.

**Offer:** on submit, every awake bird does a 300–800 ms "notice" (head turns toward the item) while the request is in flight. The resolved reaction's segments then replace the relevant plan segments. Any failure leaves the item in place with birds glancing at it. No error appears unless the user repeats the action and it fails persistently, in which case `aviary.load_failed` is shown.

### 7.6 Lighting and weather rendering

- **Lighting:** a 1D grade LUT indexed by `sun` and phase (dawn gold, midday neutral, dusk warm amber, night blue-grey at 35% luminance with a moonlit rim on sleeping birds). It is interpolated continuously and applied per cached-layer step and per bird tint.
- **Rain:** 60–140 pooled streak particles, a slight desaturation and darkening of the grade, and ripples in the pool if one is present.
- **Wind:** branch sway (a vertex offset along the branch path) and leaf gusts. Birds brace (a small crouch).
- **Reduced-motion rain:** a static "rain wash" texture cross-faded in at 30% alpha, plus the grade shift. No streaks.

### 7.7 Ambient ornaments

Leaves and feathers are generated client-side with exponential inter-arrival (mean 25–40 s), at most 3 in the scene, pooled. Birds with high `interest` may track one with a head turn (curiosity made visible). Ornaments never touch simulation state.

### 7.8 Top bar and panels

- **Top bar:** a 44 px band over the reserved sky band, with no birds under it. It holds five icon buttons with visible text labels on hover or focus and accessible names:
  1. Account & settings
  2. Accessibility
  3. Field Notebook
  4. Offer
  5. Settle
- **Fade:** after 3 s without pointer movement the bar fades to 8% opacity over 1.2 s. It returns on `pointermove`, `keydown`, focus within the bar, or a touch in the top 64 px. The fade can be disabled in accessibility settings.
- **Panels:** notebook, settings, and offer open as popovers or drawers. They are full-screen sheets below 600 px width. The scene and audio keep running behind them. Focus is trapped inside modal panels and returned to the invoking control on close.
- **Notebook panel:** paper texture, lowercase entries grouped by local date. Virtualized list (at most ~30 DOM nodes). Newest first, with infinite back-scroll through the cursor API.
- **Adoption card** (starters, candidates): naturalist voice ("two birds have come to the aviary." / "a small finch has been visiting the back perch."). One name field per bird with the suggestion prefilled. "keep" and "let it go" buttons. No catalog, no stats, no adoption dates.

### 7.9 Reduced-motion mode (a designed surface)

- **Poses:** each species has a still-pose library of about 14 poses: neutral, preen L/R, scan L/R, tilt, fluffed, sleeping, calling (beak open), drinking, bathing, looking-at-viewer, braced, alert.
- **Transitions:** the bird's current activity maps to a pose sequence. Poses cross-fade over 700–1200 ms, holding 4–12 s (mood-shaped: drowsy holds long; alert changes more often).
- **Changing perches:** a 1.2 s cross-fade out at A and in at B, with no path.
- **Greeting:** a cross-fade to looking-at-viewer or calling, and the call plays.
- **Removed:** parallax, leaves and feathers, rain streaks, and wingbeats.
- **Kept:** lighting and color shifts, slowed to 2× duration.
- **Captions, narration, and audio:** unchanged.
- **Design:** the reduced-motion art (pose sheets, cross-fade timings, rain wash) is specified by the visual designer as its own deliverable with its own review. It is a launch gate, not a derived fallback.

### 7.10 Lifecycle and memory

- **On `visibilitychange → hidden`:** stop rAF, fade audio and suspend it, flush the outbox with `sendBeacon`, and stop heartbeats.
- **On visible:** pull the snapshot, resume, and possibly greet.
- **Suspend detection:** a rAF delta > 5 s, or a divergence between `Date.now()` and `performance.now()` → treat as a resume.
- **No growth over 30 minutes:** all pools are fixed. The notebook list is virtualized with row recycling. Caption nodes come from a pool of 8. The live region's text is replaced, never appended. One `AudioContext` and one worklet node exist for the lifetime of the page. Only ≤ 2 snapshots are retained (current and previous).

---

## 8. Audio pipeline

### 8.1 Graph

```
AudioWorkletNode "syrinx" (N outputs = up to 8: 7 birds + offering voice)
   └─ per-bird GainNode (listen-in mix) ─ StereoPannerNode (x position, ±0.35 max) ─┐
Ambient bed worklet (wind/leaf noise, rain, night texture) ─ GainNode ──────────────┤
                                                         chorus bus ─ DynamicsCompressor (soft knee) ─ master Gain ─ destination
```

All nodes are created once, at audio init. Per-call state lives inside the worklet in preallocated voice slots: 12 voices, with the quietest voice stolen when all are busy.

### 8.2 Synthesis

**Note model:** each note is a pitch glide `f0 → f1` with a contour curve, duration of 30–400 ms, an amplitude envelope (attack 2–30 ms), and one of these articulations:
- **clear:** sine plus 2nd/3rd harmonics
- **buzzy:** FM with index 1–3
- **liquid:** sine with fast vibrato at 25–40 Hz
- **chip:** band-passed noise transient plus a short tone
- **churr:** AM at 30–40 Hz, the nightjar-like voice

**Motifs:** each species has 6–10 motif templates, each a sequence of note templates with ranges. A **bout** is a sequence of calls (motifs) with inter-call gaps.

**Per-call variation:** ±1.5% tempo jitter, ±20 cents per-note detune, occasional omitted or doubled notes, amplitude variation. No two calls are identical, and two same-species birds never phase-align.

**Speaker realism:** fundamental pitch floor ≥ 450 Hz, with harmonic reinforcement for the low-register species, so phone speakers carry every bird.

### 8.3 Per-bird signature (recognizable across mood and drift)

These are frozen in `voice_params` at adoption:
- a register center (species band + individual offset, allocated so every bird in an aviary is ≥ 2 semitones from any same-species bird and the aviary spans distinct bands)
- a signature motif (always the first call of a bout, e.g., Pip's rising two-note)
- a timbre fingerprint (harmonic weights, FM ratio)
- a base tempo

**Mood and traits may change only:**
- tempo (±15%)
- loudness (±4 dB)
- ornament count
- choice of non-signature motifs
- bout length and rate (vocal frequency)
- alarm-motif use (wary)

**They never change** register, signature motif, or timbre. This is enforced by the type split `VoiceSignature` (immutable) vs. `VoiceModulation`.

### 8.4 CallSpec: one source of truth

`voice.generate(signature, modulation, boutContext, rng) → CallSpec`, where `CallSpec = {bird, notes[], contour, articulation, register, loudness, repeats, pauses}`.
- The main thread schedules CallSpecs from plan `call_bout` and `respond`/`chorus` segments (and from `choreo` for greetings) and posts them to the worklet, 200 ms ahead, with the start time in AudioContext time.
- The same `CallSpec` goes to `voice.describe()` for captions (§9.3). The caption therefore describes exactly what was played.

### 8.5 Chorus mixing

- **Turn-taking:** at most 4 simultaneous calling birds. Additional bouts are deferred 200–900 ms, like real acoustic-niche behavior.
- **Joining a chorus:** joiners align loosely to the leader's tempo, quantized to ±40 ms, never sample-locked.
- **Headroom:** each bird's gain is normalized by `1/√(active_calls)`, then a soft-knee compressor (threshold −18 dB, ratio 3:1) and a limiter at −1 dBFS.

### 8.6 Listen-in mix and its decay

- **Engage:** the focused bird goes to +4 dB. The others go to −10 dB with a floor of −14 dB (never silent) and a gentle 3.5 kHz low-shelf cut, which reads as "further away." The ramp is `setTargetAtTime` with τ = 0.7 s, settling in about 2.5–3 s: slow, not a switch.
- **Disengage** (click the same bird, another bird, empty scene, Escape, focus leaves the scene): the mix returns to neutral with the same τ. Switching to another bird is one continuous rebalance, never a dip to neutral first.
- **Idle decay:** if listen-in is active and local presence lapses (the activity window expires) or the tab is hidden, listen-in disengages with the same slow ramp, and a `listen_in end` event is emitted.
- **Silent mode:** captions for the focused bird appear for every call, and other birds are captioned less often (§9.3).

### 8.7 Ambient bed

A procedural bed runs at −30 to −24 dBFS:
- filtered pink noise with slow modulation for air and leaves
- rain as noise plus sparse drop transients
- at night, a sparse, soft insect-like pulse texture, very quiet, so night is never dead air

### 8.8 Autoplay, hidden tabs, and fallback

- **Autoplay:** the AudioContext is created at init. If it is `suspended`, the aviary renders silently. A `pointerdown`, `keydown`, or `touchend` anywhere calls `resume()`, and master gain fades in over 2 s. No prompt, ever.
- **Hidden tab:** a 2 s fade, then `suspend()`.
- **Fallback triggers (permanent silent mode for the page):** no `AudioContext` or `AudioWorklet`; `resume()` rejects, or the state stays suspended after 2 gestures; `onprocessorerror`; a device error; or `statechange` to `interrupted` that doesn't recover within 10 s (iOS).
- **Silent mode:** calls are still generated and scheduled, so captions and beak motion follow them. Captions default **on** (the user can turn them off). The audio-context error counter is incremented in aggregate RUM.

### 8.9 No recorded audio, enforced

- CI scans `dist/` and fails on any `audio/*` MIME type, known audio magic bytes, or data-URI audio.
- The song-fragment library is note data.
- The worklet module is code only.

---

## 9. Accessibility surfaces

### 9.1 Semantics

- **Scene:** `role="group"` with the accessible name "aviary". It is one tab stop, with roving `tabindex` over bird elements.
- **Birds:** each is a transparent `<button>` overlay positioned over the bird's hit box (minimum 44×44 CSS px, larger than small back-perch sprites on phones), with `aria-pressed` reflecting listen-in.
  - The accessible name is the bird's name.
  - `aria-description` holds a prose description ("a small grey warbler on the front rail, preening"). It updates at most every 30 s and only on perch or activity changes.
  - Overlays are repositioned each frame by `transform` only, for 7 elements.
- **Narration:** a visually hidden `aria-live="polite"` region, with an optional visible "narration text" strip (accessibility setting).
- **Captions:** `aria-hidden="true"`, because narration covers audio for screen-reader users and we avoid double speech.

### 9.2 Narration engine (`prose.narrate`)

**Inputs:** the current snapshot, plan segments near `now`, lighting, weather, recent client-side events (greeting, offer reaction, settle), and a 10-item memory of recent clause signatures.

**Composition:** 1–3 clauses:
1. an optional scene clause (light, weather, time), included only when it has changed or every ~4th narration
2. a focal bird clause (the most salient: the listened-in bird, a bird that just called or moved, or a greeter)
3. an optional secondary clause (a relation between birds, or the quiet others)

**Naming:** the first narration after load uses descriptors with names ("pip, the small grey warbler, is on the front rail, calling softly."). Later narrations lead with names.

**Cadence:**
- idle: one narration every 45 ± 15 s
- promptly (within 1 s, still polite, never assertive): greetings, offer reactions, settle and undo, a listen-in start ("pip's calls come forward; the others soften.")
- at most 1 update per 8 s under any load, with a queue that merges events

**Setting:** narration frequency is normal, slower (90 s), or off.

**Examples of the intended register:**
- "it is early morning in the aviary; the light is thin and gold. wren is on the back perch, fluffed against the cool air."
- "pip looks up from the front rail and calls twice, softly."
- "a light rain is passing. both birds have gone quiet under the leaves."

### 9.3 Captions

- **Opt-in** from accessibility settings. **Default on** in silent mode.
- **Generation:** `voice.describe(CallSpec)` maps:
  - note count → words ("two-note", "three-note", "a run of notes", "a trill")
  - contour → "rise", "fall", "level", "rise and fall"
  - articulation → "sharp", "soft", "buzzy", "liquid", "churring"
  - register → "high", "low"
  - repeats and pauses → "…, paused, … again"
  - loudness → "soft", "clear"
  - location → "from the back perch" when the bird is not the focused one
  - Species flavor lexicon, and a 5-item dedupe that picks alternate phrasings.
- **Density:**
  - The focused bird: every call.
  - Other birds: at most one caption per bird per 20 s.
  - Chorus episodes: one group caption ("pip and wren call back and forth").
  - At most 3 captions on screen at once.
- **Placement:** above the bird's head, avoiding the top band and other captions (greedy placement over 8 candidate offsets). Fade in and out with the call, lingering 1.5 s after it.
- **Contrast:** a caption plate at ≥ 72% alpha, with text/plate polarity chosen by lighting phase. Automated screenshot contrast tests over the day × weather × mode matrix require ≥ 4.5:1.

### 9.4 Keyboard

| Key | Action |
|---|---|
| Tab / Shift+Tab | Top-bar items (in order), then the scene (one stop), then out |
| Entering the scene | Focuses the leftmost bird. The order is left-to-right by x at entry and **frozen** while focus stays in the scene, so moving birds don't reorder the list. |
| ← → ↑ ↓ | Previous / next bird. Home / End go to the first / last. |
| Enter / Space | Toggle listen-in on the focused bird |
| Escape | Exit listen-in (focus stays). In a panel, close it. |
| `o` | Open the offer menu (single-key shortcut, active only when focus is inside the app and not in a text field; can be turned off or remapped per WCAG 2.1.4) |
| Offer menu | Radio group (seed, song fragment, still pool). Song fragments are listed with prose names ("a slow falling phrase"). Enter to offer. Placement near the focused bird. |
| Settle | Top-bar button. Enter, Space, or Escape within 5 s undoes. |
| Candidate bird | Focusable like other birds. Enter opens the adoption card. |

**Focus indicator:** a 2 px dark inner ring plus a 2 px light outer ring (≥ 3:1 against any scene state), shown on `:focus-visible` only. This is the one sanctioned mark inside the scene, and it appears only for keyboard users. `prefers-contrast: more` thickens it to 3 px.

### 9.5 Contrast, forced colors, and text

- All chrome text meets WCAG AA (4.5:1; 3:1 for large text and icons).
- Top-bar icons meet ≥ 3:1 at full opacity.
- In forced-colors mode, chrome uses system colors. The canvas is left alone, and the bird overlays expose system-color focus rings.
- Text resizes to 200% without loss. Panels reflow at a 320 px width.

### 9.6 Presence equity for assistive technology

Screen readers in browse mode may swallow key events, so a screen-reader user who is listening to narration could be under-credited for presence.
- **v1:** the strict definition stays. `keydown` from navigation keys counts, and the 5 min window already favors long, still attention.
- The "screen-reader" persona in the calibration harness (activity every 4–6 min) must reach ≥ 80% of the regular persona's drift. Moderated sessions with screen-reader users in dogfood and beta check that it does.
- If it falls short, the product decision logged for that case is to widen the window, never to loosen the three-signal conjunction.

### 9.7 Accessibility test matrix and gate

- **Screen readers:** VoiceOver with Safari (macOS, iOS), NVDA with Firefox and Chrome, JAWS with Chrome, TalkBack with Chrome.
- **Automated:** axe in CI on every panel and state.
- **Manual:** a scripted keyboard-only run.
- **Reduced-motion:** visual-regression snapshots.
- **Contrast:** matrix tests.
- **Gate:** screen-reader matrix sign-off by the accessibility specialist and designer sign-off of reduced-motion mode are **launch blockers** (§14.3).

---

## 10. Voice and content system

### 10.1 Grammar architecture

- `prose` is a typed, Tracery-like grammar. Symbols are functions of a typed observation and slot fillers:
  - bird name and descriptor
  - perch name: front rail, low branch, middle perch, high branch, back perch (each slot has a naturalist name)
  - time of day, weather, activity verbs, comparative phrases
- Every template declares its inputs, so a template cannot reference data it wasn't given, such as presence.
- **Launch variety targets:**
  - notebook: ≥ 25 observation types × ≥ 8 templates each, with slot-level variation
  - narration: ≥ 300 clause templates
  - captions: a combinatorial generator plus ≥ 40 phrasing alternates

### 10.2 Authoring workflow

- A staff writer (content designer) owns the grammar, as a full team member.
- Templates live in versioned YAML with an example-output preview tool.
- Each release generates 10k samples, lints them, and the writer reviews a stratified 500-sample set before each gate.

### 10.3 Linter rules (CI-blocking for naturalist strings)

- Lowercase only, except the proper UI label "Field Notebook".
- No `!`, no "you" or "your", no second person.
- No digits, except clock times in rare notebook contexts.
- Banned lexicon: achievement, unlocked, streak, level, badge, points, score, xp, rank, welcome back, congratulations, great job, every day, days in a row, visited, you've been, missed you, record, happier, stats.
- No trait names as nouns in product prose (boldness, warmth, saturation, curiosity, vocal frequency).
- Present tense for narration and captions. Notebook entries may use the past for the day's events ("pip greeted before wren today").
- **System copy** lives in a separate registry (`copy/system.*`) linted for the *opposite* register: sentence case, no naturalist vocabulary (perch, notice, settle as a verb, and so on). The registry is the only source of error strings.

---

## 11. Privacy, security, and account lifecycle

### 11.1 Identity and PII

- The email exists only as ciphertext in `account_email` and is decrypted only inside the mailer, the settings "your email" view, and the visit-log view (for visitor emails).
- The per-account DEK is KMS-wrapped. Hard delete destroys the DEK (crypto-shred), which makes any lingering backup ciphertext unreadable.
- **Every other reference** — DB keys, shard keys, cache keys, queue messages, logs, traces, metrics labels, error reports — uses the account or aviary UUID, or nothing.

### 11.2 Logging and error tracking

- **Logs** are structured, with an allowlisted field schema. Unknown fields are dropped at the logger. A scrubber redacts any email-shaped or token-shaped string.
- **Request bodies** are never logged. Event payloads are never logged.
- **Error tracking:** self-hosted. No user context, no breadcrumbs of clicks or console, no request bodies, URL query strings stripped.
- **PII canary:** a synthetic account with a unique canary email runs through every flow nightly. A job greps Zone C (logs, traces, error events) for the canary and for its blind index. Any hit pages the on-call.

### 11.3 Telemetry boundary

- RUM goes to a separate cookieless domain. The payload schema is an allowlist of histograms and counters:
  - navigation timing, TTFB, time-to-first-bird
  - frame-time histogram, long-frame count
  - audio-context state and error counts
  - snapshot fetch latency and status class
  - JS error counts by error class
  - browser family and major version, device class, coarse country
  - bird-count bucket (D24)
- The collector drops IPs after deriving the country. There are no user, session, or aviary identifiers.
- **Server metrics:** request counts, latencies, error rates, tick lag and duration, queue depth, lease churn, and integrity counters. Labels never include account, aviary, or bird IDs.
- **Session-duration histogram:** computed client-side at `pagehide` as a bucketed duration and sent to RUM. There is no server-side per-account join.
- **Capacity:** daily unique active aviaries are counted with a HyperLogLog sketch inside Zone B, and only the count is exported.
- **Never:** third-party analytics SDKs, session replay, ad pixels, A/B frameworks that bucket accounts for behavioral analysis, or product retention cohorts.

### 11.4 Auth hardening

- Magic tokens are 32 random bytes, stored as SHA-256 hashes, with a 15-minute TTL and single use.
- The magic-link email is fixed matter-of-fact text: "Sign in to Pocket Aviary. This link expires in 15 minutes." It includes the requesting browser family, to help spot phishing.
- Sessions are opaque random tokens, revocable, with a 90-day sliding expiry on use.
- New device labels are derived from UA Client Hints (browser + OS only).
- **Email change:** a verification link is sent to the new address. The switch commits on verification. The old address is notified by matter-of-fact email ("Your sign-in email was changed…") and keeps working until the new one verifies.

### 11.5 Export and deletion

- **Export job:**
  - Builds `{ export_version, generated_at, account:{email, settings}, aviary:{created_at, birds:[{id, name, species, adopted_at, mood, simulation_state:{traits}}], notebook:[…]} }`.
  - Encrypts the object at rest, uploads it, emails a 7-day signed link, and deletes the object at expiry.
- **Soft delete:**
  - The status becomes `pending_deletion` immediately.
  - Visits are suspended. The tick keeps running, so a restored aviary has continued.
  - Every signed-in page shows the `delete.pending` panel with "I changed my mind."
- **Hard delete (day 30, daily job):** delete all Zone A and Zone B rows by account and aviary ID, destroy the DEK, purge the regional snapshot caches and pending-offer keys, and delete exports. The account UUID is appended to the **tombstone list** that restore procedures re-apply. Zone C logs age out within 14 days, and telemetry never had the ID.
- **Verification:** an end-to-end deletion test runs nightly in staging, including a restore-from-backup-then-tombstone drill each quarter.

### 11.6 Application security

- **CSP:** `script-src 'self' 'sha256-…'` for the one inline boot script. The inline snapshot is `type="application/json"` and is never executed.
- **Bird names:**
  - Validation: NFC-normalized; 1–24 grapheme clusters; letters, marks, spaces, apostrophes, and hyphens only; no control or bidi-override characters.
  - Rendering: as text only (DOM `textContent`, and canvas never renders names).
  - Visitors see host-chosen names in narration, so the same sanitization applies.
- Rate limits cover auth, invitations, exports, events, and offers. Bot protection on `/auth/magic-link` is a proof-of-work or Turnstile-class challenge, shown only when heuristics trip, with an accessible alternative.

---

## 12. Performance budgets and observability

### 12.1 Budgets

| Budget | PRD ceiling | Our internal target | Enforcement |
|---|---|---|---|
| Initial JS at first paint (gz) | < 2 MB | ≤ 90 KB (`core` ≤ 60 KB + boot + CSS) | `size-limit` in CI, per chunk. Lazy chunks: audio ≤ 60 KB, prose ≤ 45 KB, notebook ≤ 30 KB, settings/visits ≤ 70 KB, adoption ≤ 30 KB. |
| Total transferred before first bird | — | ≤ 120 KB on a cold cache | Lab test |
| Time to first bird, mid-tier mobile, 4G | < 500 ms | Lab p75 ≤ 400 ms, RUM p75 ≤ 500 ms (returning) | WebPageTest in CI nightly on a Moto G-class device profile; RUM alert |
| First-ever aviary view (after the magic link) | — | ≤ 800 ms (connection warm from the verify page) | Lab |
| Idle motion, 5-year-old laptop | 60 fps | p95 frame ≤ 16.7 ms, dropped frames < 1% over 30 min, 7 birds, rain on | Perf lab (§13.5) |
| Memory over 30 min | no growth | Heap slope ≈ 0 (regression slope < 20 KB/min; ≤ 1 MB delta at 5 vs 30 min after forced GC); DOM nodes, AudioNodes, and canvases constant | Nightly soak in CI (real time); 5-min PR smoke |
| Snapshot payload | "kilobytes" | ≤ 4 KB gz at 7 birds | Contract test |
| Tick | p99 alarm at 5 s | Tick lag p99 ≤ 2 s; compute p99 ≤ 5 ms per aviary | Alerts (§12.4) |
| Offer resolution | — | p95 ≤ 250 ms end to end | API SLO |

**Reference devices:**
- 5-year-old laptops: an i5-1135G7 / Iris Xe / 8 GB Windows laptop running Chrome, and a 2020 Intel MacBook Air running Safari.
- Mobile: a Moto G Power (2022) class device and a Galaxy A-series mid-tier device.
- 4G profile: 70 ms RTT, 12/4 Mbps (returning visitor, DNS cached).

A pessimistic "slow 4G" profile is tracked but not gated.

### 12.2 What we measure

- **Client (aggregate RUM):** TTFB, first-bird (a `performance.mark` after the frame that contains a bird at ≥ 50% opacity commits), frame-time histogram, long frames, audio-context outcomes, snapshot fetch latency and errors, JS error classes, unsupported-browser hits, and the session-duration histogram.
- **Synthetic:** a fleet of automated browsers in 6 geographies, every 10 min, using synthetic accounts. Measures first-bird, frame timing, audio init, and a scripted listen-in, offer, and settle flow.
- **Server:** tick lag and duration, dormant-batch backlog, promotions per minute, lease churn, fencing rejections, event ingest rate and rejects, offer latency, snapshot publish latency per region, cache hit rates, mailer send and bounce counts, export and deletion job success, and database health.
- **Integrity** (computed inside Zone B, exported only as counts):
  - monotonic-trigger violations
  - personality "reset" suspects (a bird whose traits equal its seeds after ≥ 7 days with presence credit)
  - event-cursor regressions
  - birds without personality rows
  - aviaries whose `last_tick` is more than 20 min behind

### 12.3 What we deliberately do not measure

- Per-account or per-bird behavior of any kind in telemetry: drift distributions, mood distributions, notebook rates, listen-in frequency, offer popularity, greeting outcomes.
- Retention cohorts, streak-like engagement KPIs, DAU/MAU per user, funnels beyond aggregate sign-in counts.
- Visitor behavior beyond the host's own visit log.

**Consequence:** drift, mood, and notebook tuning uses the simulation harness and the dogfood panel only (§14).

### 12.4 Alerts and SLOs

- **Page:**
  - tick lag p99 > 5 s for 5 min (the PRD alarm)
  - any integrity counter > 0
  - snapshot publish failures > 1% for 5 min
  - sign-in success < 97% for 15 min
  - PII canary hit
- **Ticket:**
  - RUM first-bird p75 > 500 ms for a day
  - frame p95 > 16.7 ms on the synthetic reference for 2 runs
  - audio-context failure rate > 3%
  - memory soak failure
- **SLOs:** snapshot availability 99.9%, event ingest 99.9%, offer p95 ≤ 250 ms.

---

## 13. Testing and verification

### 13.1 Engine

- **Property tests** (fast-check, 10⁵ cases): monotonicity, ceiling bounds, non-negative drive, determinism (same inputs → same bytes), and schedule equivalence (T10).
- **Invariance tests:** attunement cannot affect the listed outputs. The observer cannot read presence (type test and runtime assertion).
- **Golden replays:** 50 recorded histories.

### 13.2 Calibration suite

- Personas × T1–T10, run on every engine change.
- A diff report (trait trajectories per persona, JND crossing days) is attached to the PR.
- A 60× time-accelerated staging environment lets the team *watch* a synthetic aviary's 3 weeks in about 8 hours, for perceptual review.

### 13.3 Sync and chaos

- Kill a worker mid-transaction.
- Pause a worker past its lease, then resume it. Its write must be fenced out.
- Duplicate event batches (idempotency).
- Out-of-order and late events.
- Two devices overlapping presence.
- Revocation mid-session.
- Region failover drill in staging each quarter.
- Clock skew of ±5 min on clients.

### 13.4 API and contract

- Schema contract tests: snapshots contain no trait fields, visitor snapshots contain no host-private fields, and visitor tokens are rejected on every write endpoint.
- Mailer template allowlist.
- Rate limits.

### 13.5 Client

- **Visual regression:** deterministic seeded scenes over day × weather × mode × viewport.
- **Layout properties:** no crop, no overlap, top band clear.
- **First frame:** frame 1 contains a non-rest pose. There is no DOM element with spinner or progress semantics.
- **Greeting variety:** 1,000 samples per bird.
- **Performance lab:** 30-min scripted sessions on the reference devices with tracing.
- **Memory soak:** as specified in §12.1.
- **Cross-browser e2e** (Playwright on Chromium, WebKit, and Firefox, plus weekly real-device runs on BrowserStack-class devices): sign-in, adoption, listen-in, offer, settle/undo, notebook, visit, revocation, export, and delete/restore.

### 13.6 Audio

- **Golden-call regression:** a fixed seed renders CallSpecs through an offline worklet, then compares spectrogram similarity. A change to any existing bird's signature beyond threshold fails CI.
- **Listening panels:** before gate G2 and again before GA. N ≥ 30 participants and 7 synthetic birds. After 10 min of familiarization, identification accuracy must be ≥ 80% on unseen calls in mixed moods. A 5-point uncanniness and naturalness rating must reach median ≥ 4 and "sounds like a toy/beeps" < 15%.
- **A 30-min fatigue listen**, with no calls perceived as repeated.

### 13.7 Anti-feature audit (each gate)

A checklist review plus automated greps:
- No toast, snackbar, or banner components.
- No counters or dates of visits.
- No trait terms in UI copy.
- No announcement copy.
- No spinners.
- No audio assets.
- No analytics SDKs, verified in the lockfile and at runtime (network egress allowlist in e2e).
- No re-engagement email templates.

---

## 14. Rollout

### 14.1 Team (about 11)

- 2 client rendering and animation engineers
- 1 audio/DSP engineer
- 2 backend engineers (engine; API/auth)
- 1 infra/SRE
- 1 accessibility engineer (specialist)
- 1 staff writer / content designer
- 1 visual and motion designer (owns the reduced-motion design and the palette spec)
- 1 QA engineer
- 1 PM

A privacy counsel reviews at gates.

### 14.2 Phases (about 24 weeks to GA)

| Weeks | Phase | Exit criteria |
|---|---|---|
| 0–3 | **Foundations and spikes** | **G1:** a Canvas2D spike holds 60 fps with 7 birds plus rain on both reference laptops. If not, switch to WebGL2 now. **G2:** a 7-voice synthesis spike passes a preliminary listening panel (≥ 70%). **G3:** the edge first-frame spike reaches ≤ 400 ms in the lab. Engine skeleton with determinism and schedule-equivalence tests. Calibration harness v0 with personas. CI budgets (size, audio-asset ban, lint bans) live on day one. |
| 3–10 | **Core loop (internal alpha)** | Auth, adoption, tick with leases and fencing, snapshot pipeline (edge and regions), presence, events and outbox, layout, rigs, idle, plan playback, greetings, listen-in, audio engine and mix, settle/undo, day/night. Keyboard and focus model and the live-region skeleton land **in this phase**, not later. |
| 8–15 | **Depth** | Offers and resolver, weather and startles, social behavior and choruses, notebook observer and grammar, narration, captions, the complete reduced-motion mode, accounts (sessions, email change, export, deletion), visits, telemetry and privacy pipeline, integrity jobs. |
| 12–18 | **Dogfood** (staff and consenting friends, about 150 accounts) | This must start at least 4 weeks before beta, because the 3-week visible-drift target can only be validated in real time. A perceptual panel compares weekly still renders of their own birds, asking "does any bird seem different? how?". The success criterion: visible difference is reported by week 3–4 and not by week 1. Moderated screen-reader and reduced-motion sessions. Presence-window calibration. |
| 16–22 | **Closed beta** (waitlist, ≤ 2k accounts) | Every launch gate green for 2 consecutive weeks. Zero integrity violations. Chaos drills passed. Audio panel re-run. |
| 22–24 | **GA** | Open sign-up, throttled by a capacity-based waitlist (sign-up tokens), ramping ×2 per week while tick lag and RUM stay green. |

### 14.3 Launch gates (all must pass)

1. **Performance:** lab time-to-first-bird p75 ≤ 400 ms; 60 fps on the reference laptops; 30-min memory soak green on Chrome, Safari, and Firefox; bundle budgets.
2. **Accessibility:** screen-reader matrix signed off, reduced-motion design signed off, keyboard complete, contrast matrix green, the screen-reader persona at ≥ 80% drift parity.
3. **Drift:** T1–T10 green. Dogfood perceptual criterion met.
4. **Sync:** chaos suite green. No integrity violations in beta.
5. **Privacy:** telemetry schema review, PII canary clean for 14 days, deletion end-to-end including the backup drill, zone IAM tests.
6. **Voice:** lint clean. Writer sign-off on 500 samples each of notebook, narration, and captions.
7. **Audio:** listening panel meets §13.6.
8. **Anti-feature audit:** clean.

### 14.4 Ramping birds per aviary

Every aviary starts with 2 birds. The first real third-bird candidates appear about 85 days after an aviary is created, so production won't hold multi-bird aviaries until about month 3 of beta.
- **Before then:** internal time-travel test aviaries (staff-only environment; the age override does not exist in production builds) exercise 3–7 birds across rendering (60 fps at 7 birds plus rain plus a chorus), audio (the listening panel is run at 7), layout (narrowest phone at 7 birds), and narration and caption density.
- **Control:** arrivals are governed by `lifecycle.max_birds_effective` (engine config, default 7). If RUM frame timing in the {5–7} bucket regresses, or a 7-bird recognizability panel falls below target, lower it. Users see nothing: the next candidate simply hasn't arrived yet. Nothing already given is taken away, and existing birds are never removed.
- **Schedule:** the arrival ages are config and can be stretched, but only forward, never retroactively.

### 14.5 Instrumented from day one (first alpha build)

- Tick lag and duration.
- Integrity counters.
- Fencing rejections.
- Snapshot publish latency.
- Event ingest and rejects.
- RUM first-bird and frame histograms.
- Audio-context outcomes.
- The PII canary.
- Deletion-job health.
- Mailer outcomes.
- Bundle-size CI.
- The nightly memory soak.

### 14.6 Post-launch change policy

- **Engine constants** are versioned (`params_version`). A change is forward-only, requires a green calibration suite plus a reviewed diff report, and **never recomputes the past**.
- **Expression-mapping and art changes** blend per aviary over 7 days.
- **Species updates** create new generations for new birds only, unless golden-call and golden-render similarity pass.
- **Copy additions** go through the linter and writer review.
- **Any proposal for an engagement surface** (a streak, a badge, a notification, a "harmless" counter) is out of scope by charter and needs no debate. This paragraph is the debate.

---

## 15. Risks

| # | Risk | Likelihood / Impact | Mitigation | Tripwire |
|---|---|---|---|---|
| R1 | **Drift miscalibrated**: visible too early (a Tamagotchi feel) or never (a screensaver) | Med / High | Behavioral JNDs; T1–T10; dogfood perceptual panel from week 12; daily cap and diminishing returns; forward-only constants | Dogfood reports change by week 1, or none by week 5 |
| R2 | **We can't observe drift in production** because of the privacy rule | Certain / Med | Accept it deliberately. Calibrate before launch. The dogfood panel continues post-launch as the tuning channel. The simulation harness is the only way to evaluate changes. | — |
| R3 | **Traits saturate** after long use and the product stops "moving" | Med / Med | Squared headroom tail; per-bird ceilings; mood variety; age-based arrivals; social dynamics | T7 fails |
| R4 | **Lost or reset personality** (the silent worst case) | Low / Critical | DB trigger; single writer with fencing; `bird_trait_daily`; PITR; reset-suspect integrity counter; two-reviewer migrations; identity fields immutable | Any integrity counter > 0 |
| R5 | **Split-brain ticks or double-consumed events** | Low / High | Leases with epochs; atomic state + cursor commit; per-aviary seq in commit order | Fencing rejections spike; cursor regression counter |
| R6 | **Presence inflation or deflation** (mouse jigglers, touch devices, screen-reader browse mode) | Med / Med | 3-signal gate; clamps; max-union; daily caps; tunable window; screen-reader persona parity | Screen-reader parity < 80%; dogfood mobile users report no change |
| R7 | **Audio sounds synthetic or uncanny** (beeps, R2-D2) or fatigues | Med / High | DSP specialist from week 0; articulation palette with noise and FM; per-call variation; listening panels at G2 and GA; fatigue listens; a quiet overall level | Panel naturalness median < 4 |
| R8 | **Birds become unrecognizable** at 5–7, or after a synthesizer change | Med / High | Register allocation; frozen signatures; golden-call CI; 7-bird panels before any aviary reaches 5 | Identification < 80% |
| R9 | **Autoplay silence** makes first sessions feel dead | High / Med | Rich visual aliveness without audio; resume on first gesture with a fade; captions available; media engagement helps returning users | Aggregate audio-resumed rate low (RUM) |
| R10 | **Accessibility regressions** (live-region spam, focus loss when birds move, reduced-motion bit-rot) | Med / High | Frozen focus order; narration rate limiter; reduced-motion visual regression; an a11y engineer on the team; a11y gates each release | axe failures; a screen-reader matrix failure on the release candidate |
| R11 | **Caption contrast fails** on the dynamic scene | Med / Med | Plates with polarity by phase; matrix screenshot tests | Any matrix cell < 4.5:1 |
| R12 | **First-bird budget missed** in far regions or on cold connections | Med / Med | Edge streaming; 3 regional caches; Early Hints; tiny core; code cache; the quiet field as graceful degradation | RUM p75 > 500 ms in any region |
| R13 | **Memory growth** from audio, DOM, or closures | Med / Med | Worklet with preallocated voices; pools; virtualization; nightly 30-min soak across engines | Soak slope > threshold |
| R14 | **Magic links consumed by email scanners**, or poor deliverability | High / Med | Confirm-click on cross-browser opens; tracking disabled; SPF/DKIM/DMARC; a dedicated sending domain | Sign-in completion < 90% of requests |
| R15 | **PII leaks** into logs, traces, or errors | Med / High | Allowlisted log schema; scrubbers; self-hosted error tracking; nightly PII canary | Any canary hit |
| R16 | **Voice drift** (generic or gamified phrasing creeps in through new templates or contributors) | Med / High | Linter lexicon; writer ownership; system-copy registry separation; anti-feature audits | Linter failures; audit findings |
| R17 | **Announcement creep** (a "harmless" toast or streak added later) | Med / High | No toast primitive; import bans; the charter paragraph (§14.6); mailer template allowlist | Audit finding |
| R18 | **Tick fleet cost** at scale | Low / Med | Dormant batching (15×); a pure, cheap engine; I/O-bounded design | Cost per 1k aviaries above plan |
| R19 | **Timezone and DST oddities** (lighting jumps; travelers) | Med / Low | Canonical tz with hysteresis; 30 s lighting cross-fade; 2 h circadian slew | Dogfood reports |
| R20 | **Export turns into a stats-checking loop** | Low / Med | Raw values only in a file; no preview; 1 per 24 h; no import; product/legal review (D4) | Support reports |
| R21 | **Visit abuse** (spam invites, forwarded links) | Low / Med | Invite rate limits; fixed email text; single claim; revocation within ≤ 60 s; a visible visit log | Mailer complaint rate |

---

## 16. Open items for product (defaults already chosen; none block the build)

1. D1: settle as a fifth top-bar icon. Needs visual-designer confirmation of the glyph and its order.
2. D4: raw traits in the export. Needs legal confirmation that behavioral data-access obligations require them. If not required, drop them from the export and keep only names, moods, and the notebook.
3. D2: the audible-presence bonus. Confirm the brief's "mute" line is meant as a drift input at all. If not, set the multiplier to 1.0, which is a config-only change.
4. D12: whether claimed visits should lapse after long inactivity (for example, 12 months). Currently they don't, following the PRD.
5. D16: final art direction for candidate-bird visits (frequency and how noticeable they are).
6. Species pool naming and art. The working set is warbler, wren, finch, thrush, dove, and a nightjar-like nocturnal species, pending the visual designer and writer.

---

## Appendix A. Tunables (initial values; all server-configurable unless marked fixed)

| Tunable | Initial | Calibrate within |
|---|---|---|
| Tick cadence | 60 s | 30–120 s |
| Dormant batch interval | 15 min | 5–30 min |
| Plan horizon (hot / dormant) | 180 s / 20 min | — |
| Keepalive pull | next tick + 2–8 s | — |
| Presence heartbeat | 30 s | — |
| Presence activity window | 5 min | 3–8 min |
| Full-credit presence per day | 20 min | 10–40 min |
| Daily effective presence cap | 2.0 units (40 eff. min) | 1.5–3.0 |
| Drive filter τ_D | 3 days | 2–7 days |
| Headroom exponent | 2 | 1–2.5 |
| Attunement half-life / floor | 4 days / 0.35 | 2–7 days / 0.25–0.5 |
| Mood minimum dwell | 4 min | 2–8 min |
| Offer cooldown per bird | 4 min | 3–6 min |
| Accepted offers counted per bird per day | 3 | 2–5 |
| Greeting latency (primary) | 0.6–2.0 s | fixed by PRD intent |
| Greeting stagger | 1.2–5.0 s | — |
| No-greeting absence threshold / refractory | 30 s / 90 s | 15–60 s / 60–180 s |
| Listen-in τ / focus gain / others / floor | 0.7 s / +4 dB / −10 dB / −14 dB | — |
| Settle light shift | 4 s | 3–6 s |
| Settle undo window | 5 s | fixed (PRD) |
| Top-bar fade delay / opacity | 3 s / 8% | — |
| Narration idle cadence | 45 ± 15 s | 30–60 s (PRD) |
| Notebook refill / capacity / daily max | 0.4 per day / 2 / 1 | — |
| Rain rate / duration | 3 per week / 6–18 min | — |
| Wind gusts | 4 per day / 1–4 min | — |
| Startles | 1–3 per day | — |
| Arrival ages | ~85 / 150 / 220 / 300 / 420 days (±10) | forward-only changes |
| Magic-link TTL | 15 min | fixed (PRD) |
| Invite expiry (unused) | 30 days | fixed (PRD) |
| Soft-delete window | 30 days | fixed (PRD) |
| Raw event retention | 30 days | 14–30 days |
| Export link TTL | 7 days | — |
| Edge token TTL / revocation propagation | 24 h / ≤ 60 s | — |
| Tick p99 alarm | 5 s | fixed (PRD) |
