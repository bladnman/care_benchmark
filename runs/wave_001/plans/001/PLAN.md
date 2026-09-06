# Pocket Aviary — v1 implementation plan

Plan slot: `runs/wave_001/plans/001`. Source material: the nine files under `prd/`. This document interprets the PRD into an executable plan for a frontier engineering team; it does not restate the spec. Every place where the PRD leaves a choice open, the choice is made here and indexed in §14 (ambiguity register) so reviewers can find each judgment call in one place.

Vocabulary follows `concepts.md` exactly: bird, aviary, call, mood, personality vector, drift, presence, listen-in, offer, settle, field notebook, visit, tick. Where the plan needs an internal name that the PRD does not define (for example the bird sleep state), Appendix B maps it back to PRD language.

---

## 0. Load-bearing decisions (read these first)

These ten decisions shape everything below. Each one exists to make a PRD claim structurally true rather than true-by-policy.

1. **The tick is a pure function.** `tick(state, events_in_window, tick_time, rng(aviary_seed, tick_index)) → (state', side_effects)`. No I/O inside, no wall-clock reads, seeded randomness keyed by tick index. This is what lets "the aviary continues without the viewer" be true at any scale: warm aviaries tick live every 60 s; cold aviaries are advanced by *catch-up* ticks that are bit-identical to the ticks that would have run live. A CI property test asserts live/catch-up equivalence.
2. **Two server-side computation paths, one writer.** The **tick** (60 s cadence) is the only writer of personality vectors, moods, perches, and notebook entries. The **responder** (sub-second, on the request path) computes immediate *plans* for greetings and offer reactions so the client never waits a tick to react; it writes only to the append-only event log and a plans table, never to canonical bird state. The tick later consumes the same records, so plan and consequence always agree.
3. **The snapshot never carries the personality vector.** Clients receive a per-bird *expression profile*: quantized, named presentation parameters (perch tendency, motion register, call register, plumage render params) derived server-side from the vector. The numbers cannot be read out of the wire format, DevTools, or any client feature. The one PRD-sanctioned place the vector leaves the server is the account export.
4. **Presence is credited per account by union of intervals, clipped by server-observed liveness.** Two devices open at once earn one device's worth of presence. A device that stopped pulling snapshots stops earning presence even if its last ping claimed otherwise.
5. **Canvas 2D scene + DOM accessibility overlay.** A single canvas draws the aviary; an invisible, positioned DOM layer provides focusable bird targets, focus rings, captions, and hit-testing. Seven birds do not justify WebGL, and the DOM overlay is what makes keyboard, screen-reader, and caption surfaces first-class rather than bolted on.
6. **One pose library, two renderers.** Continuous mode interpolates between keyed poses with procedural noise; reduced-motion mode cross-fades between the same poses on a slow cadence. Reduced-motion is a second renderer over the same behavior state machine, not a flag that disables animation.
7. **AudioWorklet synthesizer with persistent voices.** Nine pre-allocated voices (seven birds plus two for the song-fragment offer and chorus spill), no per-call node or buffer allocation, seeded per-bird call signatures. Captions are rendered from the same structured call object the synth plays, so a caption always matches what sounded.
8. **One shared TypeScript `prose` package** generates notebook entries (server), screen-reader narration (client), and captions (client) from the same grammar and voice rules. The naturalist voice is one codebase, and a voice lint runs on its template corpus in CI (no "you", no exclamation, no visit-frequency language, lowercase).
9. **First bird from the HTML itself.** The CDN edge inlines the account's current snapshot and a ≤15 KB first-frame renderer into the HTML; on repeat visits a service worker draws from the last known snapshot before the network answers. The full renderer adopts the already-drawn canvas without a visual discontinuity. This is how "already in motion" survives a 4G cold load.
10. **Privacy enforced at schema boundaries, not policy.** The simulation database and the telemetry pipeline are separate systems with no shared credentials; the metrics ingest accepts only a whitelisted schema with no account, bird, or email dimensions; an OpenTelemetry processor drops any attribute that matches UUID or email patterns; logs go through a scrubber tested in CI. Email exists in exactly one encrypted column.

---

## 1. Scope

### 1.1 In v1

| Area | Ships in v1 |
|---|---|
| Accounts | Single-user accounts; email magic-link sign-in; per-device revocable sessions; verified email change; JSON export by emailed link; 30-day soft delete then hard delete |
| Aviary | One canonical aviary per account; server-side simulation tick; two starter birds from a six-species pool; cap of seven; age-gated arrivals of additional birds; naming and renaming |
| Bird engine | Hidden five-trait personality vector; monotonic drift from presence, listen-in, offers; mood system (wary, content, curious, drowsy, alert, resting); bird-to-bird responses and chorus; perch-zone choice by mood and personality |
| Interactions | Return-greeting; idle presence accounting; listen-in; offers (seed, song fragment, still pool) with per-bird cooldown; settle with five-second undo; read-only field notebook |
| Scene | One horizontal non-scrolling scene; three perch zones; local-time day/night; rare rain and wind; ambient leaf/feather drift; subtle parallax; sparse fading top bar; quiet-field loading and empty states |
| Audio | Procedural WebAudio call synthesis; per-bird recognizable signatures; real chorus mixing; listen-in re-balance; graceful silence with captions when WebAudio is unavailable |
| Sync | Multi-device via one canonical server record; append-only interaction event log; additive server-authored drift; no client-side personality state |
| Social | Read-only visit by emailed one-time invite; per-invite opt-in; revocation; 30-day invite expiry; on-demand visit log; opt-in (default off) visit notification email |
| Accessibility | Naturalist screen-reader narration; designed reduced-motion mode; call captions; full keyboard navigation; WCAG AA contrast on all user copy |
| Performance | ≤2 MB gzipped initial JS (internal target ≤300 KB on the critical path); first bird visible <500 ms on mid-tier mobile over 4G; 60 fps idle motion on a five-year-old laptop; zero memory growth over 30 minutes (CI-enforced) |
| Browsers | Last two major versions of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported-browser page otherwise |

### 1.2 Out of v1 (and the engineering consequences)

The product non-goals are absolute and are restated here only as engineering consequences, so nobody has to re-derive them:

- **No gamification**: no counters, streaks, badges, levels, milestones, calendars, or visit-frequency surfaces anywhere, including settings, exports (the export contains state, not visit history), and notebook templates. Consequence: the codebase has no query that aggregates presence by day for display; the only per-day presence computation lives inside the tick's saturation curve and is never serialized to a client.
- **No Tamagotchi mechanics**: no negative drift, no distress states, no hunger, no death. Consequence: the drift function is monotonic non-decreasing by construction (§5.4), and a property test asserts it.
- **No social network**: no profiles, follows, discovery, comments, chat, avatars, co-presence, or leaderboards. Consequence: no cross-account aggregate tables exist; visitor sessions have no write endpoint at all.
- **No notifications** except the single opt-in visit email. Consequence: there is no push subscription code, no scheduled-email system beyond transactional auth/export/visit mail.
- **No native app.** Consequence: no protocol or schema design work for native constraints; web-only assumptions (service worker, WebAudio, DOM) are allowed.
- **No announcement UI.** Consequence: the client package contains no toast, snackbar, banner, or modal-greeting component, and an ESLint rule forbids introducing one (§1.3).
- Engineering-level exclusions for v1: no WebSockets (polling at tick cadence is sufficient and simpler; §6.2), no client-side tick under any code path, no recorded audio assets of any kind, no last-write-wins path for any bird state, no email-derived identifiers, no third-party scripts on the page.

### 1.3 Product invariants enforced by tooling

| Invariant (PRD) | Enforcement in this plan |
|---|---|
| Personality is never exposed numerically | Vector absent from every client-facing schema (zod schemas in `protocol` have no numeric trait fields); contract test diffs snapshot schema against a denylist |
| Drift never decreases | Property-based test over random event streams: every trait non-decreasing across ticks |
| Only the tick writes personality | DB role for the API service lacks UPDATE on `bird_traits`; only the tick worker role has it |
| Presence requires visibility ∧ focus ∧ recent activity | Client tracker unit tests for each single-condition case; server clips intervals to session liveness |
| No "Welcome back" / no toasts | ESLint `no-restricted-imports` on any notification component; there is none to import; PR checklist item |
| Naturalist vs matter-of-fact voice split | Two copy catalogues with distinct TypeScript types; product surfaces cannot render `SystemCopy` and vice versa; voice lint on each |
| Notebook never observes the user | Template lint rejects second person and visit-frequency vocabulary; detectors have no access to presence data (they receive an aviary-only view of state) |
| Email is PII, used once | Single encrypted column; log scrubber test; metrics schema whitelist; OTel attribute filter |
| No recorded audio | Build fails if any audio file extension appears in the client bundle |
| Bundle ≤2 MB gz | `size-limit` in CI with hard fail; internal target 300 KB on critical path |
| No memory growth over 30 min | Accelerated 30-minute soak in CI (§11.5) plus nightly real-time soak on the synthetic fleet |

---

## 2. Architecture

### 2.1 System shape

```
 Browser (host or visitor)                  CDN edge                         Origin
 ┌──────────────────────────┐   HTML+inlined  ┌───────────────┐   snapshot    ┌────────────────────┐
 │ first-frame (inline JS)  │◄────snapshot────│ edge function │◄──────────────│ api service         │
 │ core renderer (canvas)   │                 │ static assets │               │  - auth/account     │
 │ a11y overlay (DOM)       │── events ──────────────────────────────────────►│  - snapshot         │
 │ audio worklet synth      │◄─ snapshots (poll @ tick cadence, ETag) ────────│  - events + responder│
 │ prose (narration/caption)│                                                 │  - notebook/visits  │
 │ service worker (shell +  │                                                 └─────────┬──────────┘
 │   last-known snapshot)   │                                                           │ Postgres (sim)
 └──────────────────────────┘                                                 ┌─────────▼──────────┐
                                                                              │ tick worker(s)      │
                                                                              │  - warm scheduler   │
                                                                              │  - catch-up on read │
                                                                              │  - daily cold sweep │
                                                                              │  - hard-delete job  │
                                                                              │  - export job       │
                                                                              └─────────┬──────────┘
                                                                                        │ whitelisted metrics only
                                                                              ┌─────────▼──────────┐
                                                                              │ ops telemetry (sep.)│
                                                                              └────────────────────┘
```

### 2.2 Services

| Service | Responsibility | Scaling |
|---|---|---|
| `api` (Node 22 / TypeScript, Fastify) | Auth, account, snapshot assembly, event ingestion, responder (greeting and offer plans), notebook reads, visit flow, export enqueue | Stateless; ≥2 replicas; snapshot p95 <150 ms |
| `tick-worker` | Runs `engine.tick` for warm aviaries every 60 s; on-demand catch-up when `api` finds a stale aviary (bounded, inline); daily cold sweep; hard-delete and export jobs | 1 active + 1 standby; per-aviary Postgres advisory lock makes concurrent workers safe |
| `edge` (CDN edge function) | Serves HTML with inlined snapshot for authenticated requests, static assets with immutable caching, unsupported-browser page | Managed |
| `mailer` (library inside `api`, provider adapter) | Magic links, email-change verification, export links, opt-in visit notifications | Transactional provider (Postmark/SES class) behind an interface |
| Postgres 16 (`sim`) | System of record: accounts, sessions, birds, traits, events, plans, notebook, invites, visits | Single primary + replica; partition `interaction_events` by month |
| Job queue | `graphile-worker` on the same Postgres (no Redis in v1) | — |
| Object storage | Export files (24-hour signed URLs) | — |
| Ops telemetry | Prometheus/OTel collector → dashboards/alerts; synthetic Playwright fleet | Physically separate from `sim`; no credentials to it |

### 2.3 Server/client boundary (the render pipeline boundary)

| Decided by server (canonical, in snapshot) | Decided by client (presentation, never uploaded as state) |
|---|---|
| Personality vector (never sent), expression profiles (sent) | Frame-level pose interpolation, breathing/scan noise, preen sequences |
| Mood per bird and `mood_since` | Which idle micro-action plays next, within the mood's action distribution |
| Perch zone and slot; relocation transitions with start time and duration | Flight path curve; cross-fade in reduced-motion |
| Weather events (kind, start, end) | Rain/wind visuals, leaf and feather ornaments, parallax |
| Call schedule for the next 90 s (bird, offset, motif seed, register) | Synthesis rendering of each scheduled call; immediate acknowledgement calls for listen-in |
| Greeting plan on return (greeter, form, stagger for responders) | Exact pose timing inside the plan |
| Offer reaction plans (per bird: reaction, delay) | Prop placement animation, optimistic prop display before the response arrives |
| Arrivals (starter fly-in, newcomer arrival) | Fly-in animation |
| `settled_at` (account-level) and per-bird cooldown expiries | Evening lighting ramp, the five-second undo window |
| Notebook entries (rendered text, immutable) | Narration and captions text (generated locally from snapshot + local events using the shared prose grammar) |
| Server time (for clock offset) | Palette by local time; top-bar fade; listen-in mix |

Rule: the client always animates *toward* server truth and never snaps. If a snapshot says a bird is on the back perch and the client has it at the front, the client plays a relocation. If a snapshot says the mood is content and the client was expressing curious, the behavior state machine finishes its current micro-action and then draws from the content distribution.

### 2.4 Tech stack and repository layout

Monorepo (pnpm workspaces, TypeScript everywhere so engine and prose code is shared between server and client):

```
packages/
  engine/      pure simulation: tick, drift, mood, perch, weather, call scheduling, greeting, offers, observer. Zero I/O. Runs in Node and browser (for the calibration bench and tests).
  prose/       naturalist grammar: notebook templates, narration, captions; voice lint rules; system-copy catalogue kept in a separate entry point with a distinct type.
  species/     six species definitions: silhouettes (path data), plumage palettes, pose libraries, call motif libraries, caption vocabularies.
  protocol/    zod schemas for snapshot, events, plans, API bodies; generated OpenAPI; the snapshot denylist test lives here.
  client/      Vite app: first-frame entry, core renderer, a11y overlay, audio worklet, presence tracker, top bar (Preact + signals for chrome only), service worker.
  api/         Fastify service.
  tick-worker/ scheduler, catch-up, sweeps, jobs.
  edge/        edge function for HTML assembly.
  bench/       drift calibration bench and listening-test tooling (synthetic cohorts, recognizability test harness).
infra/         IaC, migrations (forward-only), environment config.
```

Choices and reasons:
- **TypeScript end-to-end** because the greeting planner, offer responder, call scheduling, and prose generation must produce identical results on the server (canonical plans, notebook) and the client (narration, captions, first-frame). Duplicating that logic in a second language would be the first place canonical and presented state diverge.
- **Postgres only** for v1: rows are small, consistency matters more than throughput, and one datastore keeps the privacy boundary simple.
- **Fastify** for a small, fast HTTP layer; **Preact** (≈4 KB) for chrome; **no framework in the scene renderer**.
- **Vite** with manual chunking so the critical path is explicit (§7.2).

### 2.5 Environments

`dev` (local Postgres, mail to console), `beta` (separate database; the only environment where the arrival schedule may be accelerated and where consented research analysis is allowed, §12.6), `prod`. Engine constants are versioned config loaded at process start, never live-toggled mid-tick.

---

## 3. Data model

All identifiers are UUIDv7 generated at creation. Email is stored once, encrypted at rest with a per-environment key, on `accounts.email_enc`; a keyed blind index `email_bidx` (HMAC) supports lookup without decryption and is never used as a foreign key or log field.

### 3.1 Accounts and auth

| Table | Columns (key ones) | Notes |
|---|---|---|
| `accounts` | `id`, `email_enc`, `email_bidx`, `created_at`, `timezone` (IANA, last reported by a client), `deletion_requested_at`, `settings jsonb` (`visit_notifications: false`, `captions: 'auto'`, `reduced_motion: 'system'`, `audio: 'on'`, `narration_pace: 'normal'`) | One aviary per account in v1; `aviaries` is a separate table anyway so the join is explicit rather than implied |
| `magic_links` | `id`, `account_id` (nullable until first sign-in creates the account), `email_bidx`, `token_hash` (SHA-256 of a 256-bit random token), `expires_at` (+15 min), `consumed_at`, `requested_ip_hash` | Consumed links are invalidated by setting `consumed_at`; replay returns the matter-of-fact expired message |
| `sessions` | `id`, `account_id`, `kind` (`user` \| `visitor`), `invite_id` (visitor only), `token_hash`, `created_at`, `last_seen_at`, `ua_class` (coarse: "Safari on iPhone"), `revoked_at`, `audio_state` (last reported), `last_snapshot_at` | `last_snapshot_at` is the liveness signal used to clip presence (§5.3) |
| `email_changes` | `id`, `account_id`, `new_email_enc`, `new_email_bidx`, `token_hash`, `expires_at`, `verified_at` | Old address works until verification commits the switch |
| `exports` | `id`, `account_id`, `requested_at`, `object_key`, `expires_at`, `emailed_at` | Export file deleted after the link expires |
| `deletion_jobs` | derived from `accounts.deletion_requested_at`; hard delete runs daily for rows older than 30 days | Cascade removes every table below |

### 3.2 Aviary and birds

| Table | Columns | Notes |
|---|---|---|
| `aviaries` | `id`, `account_id` (unique), `created_at`, `seed` (64-bit, engine RNG root), `engine_version`, `tick_index` (bigint), `last_tick_at`, `state_version` (bigint, increments per tick and per canonical write), `settled_at`, `warm_until`, `weather jsonb` (active + scheduled events), `chorus jsonb` (active chorus window, if any) | Advisory lock key = `hashtext(id)` |
| `birds` | `id`, `aviary_id`, `species_id`, `name` (nullable while a newcomer is unnamed), `call_signature_seed` (64-bit, immutable), `adopted_at`, `arrived_at`, `status` (`resident` \| `newcomer` \| `released`), `order_index` | `id` is the stable identity the PRD requires; nothing ever replaces a row's `id`, `species_id`, or `call_signature_seed` (DB trigger forbids updates to those columns) |
| `bird_traits` | `bird_id` (PK), `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` (each `numeric(7,6)` in [0,1]), `updated_tick` | Only the tick-worker DB role has UPDATE; the API role has SELECT only for export assembly |
| `bird_state` | `bird_id` (PK), `mood`, `mood_since`, `mood_timer_until`, `perch_zone`, `perch_slot`, `relocation jsonb` (in-flight move: from, to, start, duration), `last_offer_reaction_at` (cooldown anchor), `listen_in_today_seconds`, `expression_cache jsonb` | Fast-timescale state, written by the tick |
| `daily_credit` | `aviary_id`, `local_date`, `presence_seconds`, `presence_seconds_muted`, `offers_accepted_by_bird jsonb`, `offers_near_by_bird jsonb` | Saturation bookkeeping for drift (§5.4). Internal only; never serialized to any client; purged after 3 days |
| `greetings` | `id`, `aviary_id`, `bird_id`, `form`, `absence_seconds_bucket`, `at`, `local_date` | Consumed by the notebook observer ("pip greeted before wren today") |

### 3.3 Events and plans

| Table | Columns | Notes |
|---|---|---|
| `interaction_events` (partitioned by month) | `seq` (bigserial, global order), `aviary_id`, `session_id`, `idempotency_key` (unique with `aviary_id`), `type`, `payload jsonb`, `client_ts`, `server_ts`, `consumed_tick` | Append-only (no UPDATE/DELETE grants except the purge job). Retained 30 days after consumption; nothing is ever recomputed from it |
| `plans` | `id`, `aviary_id`, `kind` (`greeting` \| `offer_reaction` \| `arrival`), `payload jsonb`, `created_at`, `consumed_tick` | Responder output; returned to the client immediately and applied by the next tick |
| `presence_intervals` | `aviary_id`, `session_id`, `start_ts`, `end_ts` | Materialized from `presence.ping` events after clipping; the tick reads the union per window |

### 3.4 Notebook, visits, weather

| Table | Columns | Notes |
|---|---|---|
| `notebook_entries` | `id`, `aviary_id`, `occurred_at`, `local_date`, `detector`, `params jsonb`, `text` (rendered at creation, immutable), `salience` | Read-only forever; cursor pagination by `(occurred_at, id)` |
| `notebook_budget` | `aviary_id`, `tokens` (numeric), `last_refill_at` | Sparsity governor state (§5.11) |
| `invites` | `id`, `aviary_id`, `visitor_email_enc`, `visitor_email_bidx`, `token_hash`, `created_at`, `expires_at` (+30 d), `consumed_at`, `revoked_at` | One-time link; consumption creates a visitor session bound to `invite_id` |
| `visits` | `id`, `invite_id`, `started_at`, `last_seen_at` | Duration shown rounded to 5 minutes ("approximate") |
| Weather | lives in `aviaries.weather` | Scheduled deterministically from the aviary RNG stream, so it is reproducible during catch-up |

### 3.5 Invariants and retention

- Bird identity: `birds.id`, `species_id`, `call_signature_seed`, `arrived_at` are immutable (trigger). A migration test asserts that every bird's `id` and trait values are unchanged across migrations (§11.6).
- Personality: written only by the tick; never derived from events; the event log is purged on a 30-day schedule precisely because nothing depends on replaying it.
- Presence bookkeeping (`daily_credit`, `presence_intervals`) is purged after 3 days; it exists only to compute the saturation curve.
- Visitor data: `visits` and `invites` are the host's data and are deleted with the host's account. Visitor sessions never link to a visitor *account* even if the visitor has one.
- Hard delete removes rows in every table above, cancels queued jobs, and deletes export objects. Database backups are retained 14 days, so deleted data leaves backups within 44 days of the request; the privacy policy states this.

---

## 4. API surface

### 4.1 Conventions

- JSON over HTTPS; cookies for session (`HttpOnly; Secure; SameSite=Lax`), CSRF token double-submit on state-changing requests; CSP with a per-response nonce for the inline first-frame script; no third-party origins.
- Every account reference is the UUID. No endpoint accepts or returns an email except the sign-in, email-change, invite, and visit-log surfaces that exist to handle addresses.
- Errors return `{ code, message }` where `message` is matter-of-fact system copy from the `SystemCopy` catalogue (§9.6). The client never composes error text.
- Rate limits are per account UUID or per `email_bidx` for pre-auth endpoints; responses never reveal whether an email has an account.

### 4.2 Auth and account

| Method and path | Behaviour |
|---|---|
| `POST /auth/magic-link {email}` | Always `202`. Creates a `magic_links` row (15-minute expiry) and emails the link. Limits: 5 per address per hour, 20 per day, 30 per IP per hour |
| `GET /auth/callback?token=` | Verifies hash, expiry, and single use; creates the account on first sign-in (with two starter birds pending adoption); issues a session cookie; redirects to `/`. Expired or used: renders "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `POST /auth/sign-out` | Revokes the current session |
| `GET /account` | Settings, session list (`ua_class`, `last_seen_at`, current flag), deletion state |
| `PATCH /account/settings` | `timezone`, `visit_notifications`, `captions`, `reduced_motion`, `audio`, `narration_pace` |
| `DELETE /account/sessions/{id}` | Revoke a device session |
| `POST /account/email-change {new_email}` → `GET /account/email-change/verify?token=` | Verification to the new address; switch commits only on verify |
| `POST /account/export` | `202`; job renders JSON (birds, names, species, current personality vectors, current moods, notebook entries, settings) to object storage and emails a 24-hour link to the verified address |
| `POST /account/delete` / `POST /account/restore` | Soft delete sets `deletion_requested_at`; any signed-in page during the 30 days shows the restore affordance ("I changed my mind") in system voice |

### 4.3 Aviary state: the snapshot

`GET /aviary/snapshot?reason=initial|visible|keepalive|gap&tz=<IANA>` with `If-None-Match: <state_version>`.

Server behaviour: update `sessions.last_snapshot_at` and `accounts.timezone`; if `aviaries.last_tick_at` is more than one tick stale, run catch-up inline (bounded; §5.2); if `reason ∈ {initial, visible}` and absence ≥ 30 s, run the greeting planner and clear `settled_at`; assemble and return. `304` when `state_version` matches and no greeting plan is pending.

Snapshot shape (abridged; full zod schema in `protocol`):

```json
{
  "state_version": 48213,
  "server_time": "2026-09-06T14:03:12.410Z",
  "next_pull_after_ms": 60000,
  "absence_seconds": 5820,
  "aviary": {
    "settled_at": null,
    "weather": { "active": {"kind": "rain", "start": "...", "end": "..."}, "upcoming": [] },
    "chorus": null
  },
  "birds": [{
    "id": "0191...", "name": "pip", "species": "rufous_warbler", "status": "resident",
    "perch": {"zone": "front", "slot": 1},
    "relocation": null,
    "mood": "content", "mood_since": "...",
    "expression": {
      "perch_tendency": "forward",          // back | mid | forward | close   (from boldness)
      "warmth_register": "greets_first",     // reserved | responsive | greets_first | companionable (social warmth)
      "call_register": {"rate": "frequent", "chorus": "eager"},           // (vocal frequency)
      "plumage": {"saturation_step": 6, "detail_tier": 2},               // 0–9 steps, 0–3 tiers (plumage saturation)
      "curiosity_register": "investigates" // ignores | glances | investigates | explores
    },
    "call_signature": {"seed": "9f2a...", "base_pitch_step": 3, "rhythm_id": 4, "timbre_id": 1},
    "call_schedule": [{"at_ms": 4200, "seed": "...", "phrase": "contact", "intensity": 0.6}],
    "cooldown_until": null
  }],
  "greeting": {
    "greeter": "0191...", "form": "approach_and_call", "start_ms": 900,
    "responders": [{"bird": "0192...", "delay_ms": 3400, "form": "answer_call"}],
    "seed": "..."
  },
  "arrivals": []
}
```

Notes: expression parameters are coarse steps (at most ten levels) chosen so that a step change is a perceptible rendering change; this is also what makes the three-week "visible drift" target testable (§5.4.4). `call_schedule` covers 90 s so a late pull never leaves silence. `absence_seconds` and `greeting` are present only for `initial`/`visible` pulls by the host, never for visitors.

### 4.4 Interaction events

`POST /aviary/events` with a batch: `[{ key, type, client_ts, payload }]`. Each `key` is a client UUIDv7 idempotency key; duplicates are acknowledged, not re-applied. Response: `{ accepted: [key…], plans: [...] }` where `plans` carries any responder output (offer reactions) so the client can play them immediately.

| Type | Payload | Server handling |
|---|---|---|
| `session.start` | `{ timezone, ua_class, audio_state, reduced_motion }` | Updates session and account timezone |
| `presence.ping` | `{ start_ts, end_ts, audio_state }` (≤30 s interval, all three presence conditions held throughout) | Clipped (§5.3) and materialized into `presence_intervals` |
| `listen_in.start` / `listen_in.end` | `{ bird_id }` | Interval per bird; open intervals auto-close when the session's liveness lapses |
| `offer.place` | `{ kind: seed\|song\|pool, fragment_id? }` | Aviary-level debounce 45 s; responder computes per-bird reactions and returns a plan (§5.10) |
| `settle.begin` / `settle.undo` | `{}` | Sets/clears `settled_at`; undo accepted only within 5 s of begin |
| `session.end` | `{}` (via `sendBeacon` on `pagehide`) | Closes open intervals |

Visitor sessions receive `403` on this endpoint. There is no event type for anything a visitor could do.

### 4.5 Notebook

`GET /aviary/notebook?before=<cursor>&limit=30` → entries newest first with `local_date` for weekday headers. No write endpoints exist.

### 4.6 Birds and adoption

| Method and path | Behaviour |
|---|---|
| `GET /aviary/birds` | Names, species, status; no traits |
| `POST /aviary/adopt-starters { names: [a, b] }` | First-run only; names the two pending starters; schedules staggered fly-in arrivals (2–5 s apart) as `arrival` plans |
| `PATCH /aviary/birds/{id} { name }` | Rename; no effect on any other column |
| `POST /aviary/birds/{id}/release` | Newcomer only (`status = newcomer`); marks `released`; the bird leaves quietly on the next tick. Residents cannot be released in v1 |

Newcomer arrival and naming are described in §5.12.

### 4.7 Visits

Sequence:

1. Host: `POST /visits/invites { email }` → creates `invites` row (30-day expiry, hashed one-time token), emails the visitor a link. Response is the invite id and expiry only.
2. Visitor: `GET /visit/accept?token=` → validates, sets `consumed_at`, creates a `visitor` session cookie bound to `invite_id` (sliding 30-day expiry), redirects to `/visit`.
3. Visitor client: `GET /visit/snapshot?reason=` → same assembly as the host snapshot (including catch-up so the aviary is live) minus `greeting`, `absence_seconds`, `cooldown_until`, and any settings; records `visits.last_seen_at`. No presence, no listen-in, no offers; the client for visitors is built from the same renderer with the interaction layer compiled out by a build flag, and the server rejects writes regardless.
4. Host: `GET /visits/log` → invites (outstanding, with expiry) and visits (visitor email, date, approximate duration) newest first. `DELETE /visits/invites/{id}` revokes; the visitor's next snapshot returns `410` with "This visit is no longer available." Expired or revoked links return the same page.
5. Opt-in notification: if `settings.visit_notifications` is true, one email per visit start, rate-limited to one per visitor per hour. Off by default; not mentioned in onboarding.

### 4.8 Error surfaces and voice

All API error messages come from the system-copy catalogue and are sentence-case English:

- Sign-in failures: "We couldn't sign you in. The link may have expired. Try requesting a new link."
- Session expiry (401 on snapshot): "Your session timed out. Sign in again to keep watching."
- Snapshot failure (5xx after retries): "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- Visit revoked/expired (410): "This visit is no longer available."
- Unsupported browser: "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge."

The product surface never shows these strings inside the scene; they render in a plain system panel above the quiet field.

---

## 5. Simulation engine design

The engine is the `engine` package: pure TypeScript, no I/O, no `Date.now()`. Every function takes explicit time and an explicit RNG stream. The tick worker and the API responder are thin shells around it.

### 5.1 Tick contract

```
tick(input: {
  aviary: AviaryState,          // traits, moods, perches, weather, chorus, settled_at, budget
  events: InteractionEvent[],   // events with server_ts in (prev_tick_time, tick_time]
  presence: Interval[],         // clipped, unioned presence for the same window
  tick_time: Instant,           // = aviary.created_at + tick_index * 60 s
  tz: IANAZone,
  rng: Rng                      // seeded from (aviary.seed, tick_index); sub-streams per purpose
}): {
  aviary: AviaryState,          // new canonical state, state_version + 1
  notebook: NotebookEntry[],    // zero or more entries to insert
  greetings_consumed: [...],    // bookkeeping
}
```

Rules:
- Tick times are on a fixed grid from aviary creation, so `tick_index` fully determines `tick_time`. A tick with `tick_index ≤ aviary.tick_index` is a no-op (idempotent).
- Randomness comes from a counter-based generator (PCG-class) seeded with `hash(aviary.seed, tick_index, purpose)`. Consumption order inside a tick is fixed by code, so replays are identical.
- `tz` is the account's last-reported timezone; time-of-day features are computed from `tick_time` in that zone.
- The tick writes `aviaries`, `bird_state`, `bird_traits`, `daily_credit`, `notebook_entries`, `notebook_budget` in one transaction under the aviary advisory lock.

Per-tick order of operations: (1) ingest events → per-bird signals; (2) presence and listen-in credit → drift deltas; (3) weather schedule advance; (4) mood pressures and transitions; (5) perch and relocation decisions; (6) arrivals and releases; (7) call schedule for the next 90 s (including responses and chorus windows); (8) expression profiles; (9) notebook observer; (10) settle expiry.

### 5.2 Scheduling: warm, cold, catch-up

- **Warm** aviary: a host or visitor session pulled a snapshot in the last 10 minutes, or unconsumed events exist, or an in-flight transition (relocation, arrival, weather boundary) ends within the next tick. The scheduler enqueues a tick for every warm aviary every 60 s (`warm_until` column; scheduler query is an index scan).
- **Cold** aviary: everything else. Not ticked on a schedule.
- **Catch-up**: when `api` serves a snapshot and `last_tick_at` is more than one interval old, it runs all missing ticks inline before assembling the snapshot. Each missing tick sees an empty event window (cold aviaries have no events by definition, except events that arrived just before this read, which fall into the final tick). Bounded cost: ≤ 1,440 ticks (one day) because of the sweep below; measured target ≤ 25 µs per empty tick → ≤ 40 ms worst case.
- **Daily cold sweep**: the tick worker advances every cold aviary that is more than 24 h behind, spread across the day (hash-bucketed), so no aviary is ever more than ~25 h behind and inline catch-up stays bounded. The sweep also runs the notebook observer, so an absent user's aviary accumulates the sparse "it kept going" entries the governor allows (§5.11).
- Equivalence guarantee: for a given `engine_version`, ticking live and ticking by catch-up produce identical state. CI runs a property test generating random event streams and comparing per-minute live ticks with a single catch-up over the same span (§11.2).
- Engine upgrades: `engine_version` is stamped on the aviary; catch-up after an upgrade uses the new engine for the missing span. This is the one intentional non-equivalence; calibration changes are small and never re-run history, so it is unobservable.

### 5.3 Presence accounting (server side)

The client only reports intervals during which all three conditions held (§7.7). The server treats those reports as claims and clips them:

1. Interval must lie within `[session.created_at, server_now + 5 s]` and be ≤ 35 s long (pings are emitted every 30 s).
2. Interval is credited only if the session pulled a snapshot within the previous 180 s (liveness). A tab that stopped pulling (suspended laptop) cannot keep earning presence on stale pings.
3. Intervals are unioned per aviary across sessions before crediting. Two devices watching simultaneously earn ≤ 60 s per tick window.
4. `audio_state` on the ping marks the interval as muted or audible for the vocal-frequency modulation (§5.4.2).

Visitor sessions produce no presence.

### 5.4 Drift function

#### 5.4.1 Shape

Per trait `t ∈ [0,1]`, per tick:

```
Δt = (1 − t)^1.5 · Σ_inputs k_input · Δcredit_input
t' = min(1, t + Δt)
```

- `(1 − t)^1.5` is the saturation term: growth slows as a trait approaches its ceiling, so traits never hit hard walls and late drift is gentle.
- `Δcredit_input` is the *increment* in a per-day saturating credit curve, which is the low-pass filter the PRD asks for. For presence: `credit(P) = 1 − exp(−P / τ_p)` with `P` the unioned presence seconds so far in the local day and `τ_p = 90 min`. The tick applies `credit(P_after) − credit(P_before)`. Fifteen minutes earns 15 % of the daily maximum; an hour earns 49 %; five hours earns 96 %. A marathon session cannot buy much more than an hour of attention, and a click cannot buy anything.
- All `k` are ≥ 0 and all credits are non-decreasing, so `Δt ≥ 0` always. Neglect contributes zero. There is no term that can be negative; this is the monotonic-toward-expressive rule as arithmetic.

#### 5.4.2 Inputs and initial coefficients (per day at full credit)

| Input | Credit curve | boldness | social warmth | vocal freq. | plumage sat. | curiosity |
|---|---|---|---|---|---|---|
| Presence (aviary-wide) | `1 − e^(−P/90min)` | 0.030 | 0.025 | 0.030 (× 0.5 while muted) | 0.045 | 0.020 |
| Listen-in on bird *b* | `1 − e^(−L_b/20min)` per bird | — | 0.020 | 0.020 | — | — |
| Offer accepted by *b* | 0.004 each, ≤ 3 per bird per day | — | — | — | — | 1.0 (i.e. +0.004·(1−t)^1.5 per accept) |
| Offer near *b* (front/middle perch at offer time) | 0.002 each, ≤ 3 per bird per day | 1.0 (+0.002·(1−t)^1.5) | — | — | — | — |
| Settle | none (mood-only) | — | — | — | — | — |

"× 0.5 while muted" implements the brief's "whether you mute the calls or let them play": presence while calls are muted still drifts everything, but the vocal-frequency component at half rate. It never subtracts (ambiguity register A7).

#### 5.4.3 Seed values at adoption

Species define a base; each bird gets a seeded jitter so two birds of the same species differ:

| Trait | Starter range | Rationale |
|---|---|---|
| boldness | 0.25–0.45 | Starters begin reserved; there is room to come forward over months |
| social warmth | 0.30–0.50 | One starter is biased +0.08 so a "greets first" bird exists from day one |
| vocal frequency | 0.30–0.55 (nightjar species 0.45–0.65) | Enough calls to be audible; room to grow into chorus |
| plumage saturation | 0.20–0.35 | The most visible long-run drift channel starts low deliberately |
| curiosity | 0.30–0.50 | Offers should get a reaction from one bird early on |

#### 5.4.4 Calibration targets as tests

"Regular visits" is defined for testing as five 15-minute sessions per week with one listen-in per session. Bench cohorts (`packages/bench`): *regular* (as defined), *weekend-only* (two 40-minute sessions), *marathon* (one 6-hour session per week), *daily-glance* (seven 2-minute sessions), *abandoned* (two weeks regular, then nothing), *always-open* (tab visible but unfocused all day; must produce zero drift).

Assertions:
- After 7 days, *regular* moves at least one trait by ≥ 0.02 (instrument-measurable) and no trait by more than 0.05.
- After 21 days, *regular* crosses at least one expression step (e.g. `plumage.saturation_step` 2→3 or `perch_tendency` mid→forward): the user-visible threshold.
- *Marathon* ≤ 1.6 × *regular* over 21 days (saturation works).
- *Abandoned*: no trait decreases; expression profile at day 28 equals day 14.
- *Always-open*: exactly zero drift.
- Drift never exceeds 0.25 on any trait in 90 days for any cohort.

The expression profile step boundaries are chosen so one step is a just-noticeable rendering change (plumage: ~8 % chroma; perch tendency: ~10 percentage points of front-perch occupancy; call rate: ~25 % more calls per hour). Those JNDs are validated in the beta a/b viewing sessions (§12.3).

### 5.5 Mood

Moods: `wary`, `content`, `curious`, `drowsy`, `alert`, `resting` (the night sleep state; Appendix B). Each bird has `mood`, `mood_since`, and `mood_timer_until` (dwell). Dwell is sampled at entry: 8–40 minutes depending on mood (wary and alert are shorter; content and drowsy longer; resting lasts until the time-of-day prior releases it).

Each tick computes a pressure vector `π` over moods and, when the dwell has expired or a strong event arrived, samples the next mood from `softmax(π / 0.35)` with the tick RNG; otherwise the mood persists. Pressure sources (additive, clamped to [−2, 2]):

| Source | Effect |
|---|---|
| Time of day (account tz) | 05–08: alert +1.0, curious +0.6; 08–17: content +0.8, curious +0.4; 17–20: content +0.5, drowsy +0.7; 20–05: resting +1.8, drowsy +0.6. Nightjar species inverts: resting mid-day, alert/content at night |
| Weather | Rain: content +0.4, drowsy +0.4, vocal damping (§5.8); wind: alert +0.8 if boldness ≥ 0.5 else wary +0.8. Effects decay linearly over 10 min after the event ends |
| Recent interactions (last 15 min) | Offer accepted: content +0.9; listen-in active on this bird: curious +0.5, content +0.3; offer refused while wary: wary +0.3; settle in effect: drowsy +0.8, resting +0.4 |
| Other birds | Each wary bird within the same or adjacent zone: wary +0.4 × this bird's social warmth; an active chorus: content +0.3 |
| Personality | wary −1.2 × boldness; curious +0.8 × curiosity; alert +0.4 × vocal frequency |
| Strong events (bypass dwell) | Another bird's alarm call → wary evaluation now; user's offer → immediate evaluation for reacting birds; settle → immediate evaluation |

Mood persists across sessions by construction: it is a row, updated only by ticks (live or catch-up), and no code path resets it on session start.

### 5.6 Perch zones and relocation

Zones: `front` (3 slots), `middle` (3 slots), `back` (4 slots); ten slots for at most seven birds. Each tick, each bird has a relocation hazard (baseline 1 per 20 min; × 2.0 curious/alert, × 0.3 drowsy, × 0 resting, × 0.5 during rain). When a relocation fires, the target zone is sampled from a distribution shaped by boldness and mood:

| Mood \ boldness | low (< 0.4) | mid | high (> 0.6) |
|---|---|---|---|
| wary | back 80 / mid 20 | back 55 / mid 40 / front 5 | back 30 / mid 50 / front 20 |
| content | back 40 / mid 45 / front 15 | back 20 / mid 45 / front 35 | back 10 / mid 35 / front 55 |
| curious | back 30 / mid 45 / front 25 | back 15 / mid 40 / front 45 | mid 30 / front 70 |
| alert | mid 60 / back 40 | mid 60 / front 40 | front 70 / mid 30 |
| drowsy | back 60 / mid 40 | mid 60 / back 40 | mid 70 / back 30 |

Slot choice within a zone prefers a slot adjacent to another bird with probability `0.3 + 0.6 × social_warmth`. Relocation is recorded as `{from, to, start: tick_time + random offset within the minute, duration: 1.2–2.4 s}` so all devices animate the same move at the same server time. Resting birds do not relocate; at dawn a settled bird relocates once toward its daytime tendency.

### 5.7 Weather

Scheduled from the aviary RNG on a rolling seven-day horizon: rain events Poisson mean 2.5 per week lasting 3–8 minutes; wind events mean 4 per week lasting 2–5 minutes; never both at once; none during 00–05 local (the user would never see it and it would only dampen the nightjar). Weather is state in the snapshot, so host and visitor see the same rain at the same server time. Effects: during rain, aviary call rate × 0.4 and mood pressure as above; during wind, alert/wary pressure and slightly higher relocation hazard for bold birds.

### 5.8 Call scheduling and call-grammar runtime

The engine schedules *when* and *what kind*; the client synthesizes *how it sounds* (§8). Per tick the engine produces a 90-second schedule (overlapping the next tick by 30 s; the client discards duplicates by seed).

Per-bird spontaneous rate (calls per minute):

```
λ_b = (0.3 + 1.2 · vocal_b) · mood_mult · tod_mult · weather_mult · settle_mult
mood_mult:   content 1.0, curious 1.1, alert 1.3, wary 0.5, drowsy 0.3, resting 0 (nightjar at night: 1.2)
tod_mult:    dawn 1.4, morning 1.0, midday 0.8, dusk 0.8, night 0 (nightjar: 1.0 at night)
weather:     rain 0.4, wind 0.8
settle_mult: 0.3 while settled
```

Calls are sampled as a Poisson process per bird with a refractory gap of 6 s, then post-processed: (a) **responses**: when bird *a* calls, each other bird *b* answers within 1–4 s with probability `(0.15 + 0.5 · warmth_b) · mood_mult_b`, capped at one responder per call; (b) **chorus**: if two or more birds with vocal ≥ 0.5 call within a 10-second window, a chorus window of 20–40 s opens during which all birds with vocal ≥ 0.4 have rate × 2 and response probability × 1.5; (c) **calm cap**: total aviary calls ≤ 6 per minute (excess dropped, lowest-intensity first), so a seven-bird aviary stays a place, not a soundscape; (d) **phrase selection**: each call gets a phrase type (`contact`, `answer`, `alarm` (rare, wary only), `chorus`, `night`, `greeting`) and a seed; the client's grammar expands phrase + species motif library + bird signature + mood into the actual call.

`alarm` calls are rare (wary birds, ≤ 1 per hour) and are the source of "one bird's alarm shifts nearby birds toward wary" (§5.5).

### 5.9 Greeting planner (responder path)

Runs inside the snapshot request when `reason ∈ {initial, visible}` and `absence_seconds ≥ 30`. Output is a plan the client executes starting at `first_frame + start_ms` (`start_ms` sampled 600–1,500 ms so the greeting lands "within the first second or two").

1. **Greeter selection**: `score_b = 0.5·boldness + 0.3·warmth + mood_mod + jitter`, where `mood_mod` = curious/alert +0.15, content +0.05, drowsy −0.2, wary −0.3, resting −1.0; `jitter ∈ [−0.1, 0.1]` from the plan seed. The greeter is the argmax. If every bird is resting and a nightjar is present, the nightjar greets with a soft `night` call; if none is present, the top-scoring bird gives a minimal greeting (eyes open, weight shift). A greeter always exists; the PRD promise is "one bird notices."
2. **Form by absence bucket and greeter boldness**:

| Absence | low boldness | mid | high |
|---|---|---|---|
| 30 s – 10 min | glance up | glance + head-tilt | head-tilt + quiet two-note call |
| 10 min – 6 h | head-tilt | head-tilt + short call | step toward front + short call |
| 6 h – 3 d | short call from current perch | step forward + call | approach front perch + call |
| > 3 d | longer call, then step | approach front + longer call | approach front + longer call + second look |

3. **Responders**: each other bird answers with probability `0.6 × warmth × mood_mult`, delay 1.5–6 s after the greeter, minimum 800 ms between any two greeting actions (no unison). A wary bird's "answer" may be just a glance. At most two responders.
4. **Variation**: the form is a sequence of action motifs (glance, tilt, step, hop, fluff, call phrase, pause) assembled by the grammar with the plan seed; the call inside uses phrase `greeting`. Seeds derive from `(bird_id, session_id, state_version)`, so the same bird greets in the same *manner* across days and never with the same *sequence*.
5. The plan is stored in `greetings` (greeter, form, bucket, local date) for the notebook observer.

First session: starters fly in (arrival plans); the bolder starter greets 2–4 s after landing.

### 5.10 Offer responder

`offer.place` is handled on the request path so the reaction begins within one network round-trip:

1. Debounce: an offer within 45 s of the previous one is accepted as an event but yields an empty plan (no reaction, no drift). One prop is active at a time; a new kind replaces the old (seed swept, pool placed).
2. For each bird not in cooldown (`last_offer_reaction_at + 5 min`), sample a reaction from a table keyed by offer kind × mood × curiosity register:

| Offer | curious/content, curiosity ≥ 0.45 | content, lower curiosity | wary | drowsy | alert |
|---|---|---|---|---|---|
| seed | approach and peck (delay 2–6 s) | approach after 8–20 s | watch, then come near after 30–90 s (no peck) | no approach; glance | approach quickly, one peck, retreat |
| song fragment | join (call against the fragment's contour) | quiet listening, head-tilt | go quiet | none | call against sharply |
| still pool | bathe (bold) or drink (mid) | drink | watch from perch | none | drink briefly |

3. Reactions that count as "accepted" (peck, join, drink, bathe) set the bird's cooldown and are recorded in the event payload so the tick applies the curiosity credit and content pressure. Birds on the front or middle perch at offer time are "near" for the boldness credit.
4. The plan is returned in the `events` response and stored in `plans`. The client shows the prop immediately (optimistic) and plays reactions on receipt; if the response is lost, the prop fades after 30 s.
5. Song fragments are five procedural motifs in a "hummed" timbre from the same synth; birds join by calling in the fragment's contour class (rise, fall, arch, level, two-part).

No cooldown state is displayed. The offer affordance is always available; the engine decides what happens.

### 5.11 Notebook observer

Runs in the tick. A set of **detectors** reads an *aviary-only view* of state and history (no presence, no session data; the type system enforces this by passing a projection that lacks those fields) and yields candidate observations with a salience `s ∈ [0,1]`. A **sparsity governor** decides which are written.

Detectors (initial set of ~20; each has ≥ 6 template variants and slot-fill):

| Detector | Trigger | Salience |
|---|---|---|
| first greeter differs from the week's pattern | `greetings` for the local day vs prior 6 days | 0.85 |
| quiet stretch | ≥ 40 min with ≤ 1 call during daytime | 0.5 |
| long preen | a content bird on the same perch, preen-heavy, ≥ 20 min | 0.4 |
| unusual perch | a bird in a zone it occupies < 10 % of the time this fortnight | 0.6 |
| rain passage with reaction | weather event + at least one bird moved during it | 0.55 |
| chorus | first chorus of the day or first chorus with a newly arrived bird | 0.6 / 0.9 |
| night caller | nightjar called after 23:00 local | 0.45 |
| bath / first bath | pool reaction `bathe` / first ever for that bird | 0.5 / 0.9 |
| newcomer arrival / named / moved on | arrival, naming, release | 0.95 / 0.7 / 0.7 |
| dawn hush | no calls between civil dawn and +20 min, then a first call | 0.4 |
| ambient texture (leaf, light) | no state trigger; attached as a clause to other entries | — |

Governor: a token bucket per aviary with capacity 2.0, refill 1 token per 72 h while the aviary has had presence in the last 7 days, 1 per 168 h otherwise. A candidate is written if `tokens ≥ 1` and `s ≥ threshold(tokens)` where `threshold = 0.35` at a full bucket rising to `0.7` near empty; candidates with `s ≥ 0.85` may overdraw to −1. Hard caps: at most one entry per local day except arrivals; at least 36 h between entries with `s < 0.6`. Expected rate: one entry per 2–4 days for regular users, ~1 per week for sparse users, and a small handful across a month of absence.

Rendering: templates are lowercase, present tense, bird-named, one to three sentences; a weekday word prefixes the first entry of a local day ("tuesday — …"). Text is rendered once and stored; the observer never rewrites history. The voice lint (§9.6) forbids second person, exclamation marks, "welcome", "streak", "visited", "day(s) in a row", numbers with units of the user's behaviour, and any trait name.

### 5.12 Arrivals: starters and newcomers

- **Starters**: species chosen by the aviary RNG with the constraint that the two differ and at most one is the nightjar. Default name suggestions come from a per-species list; the adoption surface (product voice) shows both birds' descriptions and name fields. On submission, arrival plans fly the birds in 2–5 s apart.
- **Newcomers**: available by aviary age only. Schedule (days since `aviaries.created_at`, ± 15 % jitter from the RNG): third bird at 75, fourth at 150, fifth at 240, sixth at 330, seventh at 450. On the scheduled tick (morning local time), a newcomer of a species not already over-represented arrives on the back perch, unnamed, as a full bird (traits, mood, calls, drift) whose name is rendered as a species description ("the small brown one") until the user names it. The naming affordance appears when the user focuses the newcomer (listen-in panel) and in bird settings; releasing is available in the same place ("let it move on"). No prompt, banner, or notification announces the arrival; the notebook writes it. Ambiguity register A3.

### 5.13 Expression profiles

Computed at the end of each tick and cached in `bird_state.expression_cache`:

| Profile field | Source trait | Steps |
|---|---|---|
| `perch_tendency` | boldness | back < 0.3 ≤ mid < 0.5 ≤ forward < 0.7 ≤ close |
| `warmth_register` | social warmth | reserved < 0.35 ≤ responsive < 0.55 ≤ greets_first < 0.75 ≤ companionable |
| `call_register.rate` | vocal frequency | sparse < 0.35 ≤ occasional < 0.55 ≤ frequent < 0.75 ≤ voluble |
| `call_register.chorus` | vocal frequency | reluctant < 0.5 ≤ eager |
| `plumage.saturation_step` | plumage saturation | `floor(t × 10)` (0–9) |
| `plumage.detail_tier` | plumage saturation | 0–3 at 0.25 boundaries (feather detail layers) |
| `curiosity_register` | curiosity | ignores < 0.3 ≤ glances < 0.5 ≤ investigates < 0.7 ≤ explores |

Hysteresis of 0.01 at each boundary prevents flicker between steps. The client uses these words, never numbers, and the words never surface in the UI either; they drive rendering only.

### 5.14 Engine versioning and the calibration bench

- `engine_version` is a semver stamped per aviary at each tick. Constants live in `engine/constants.ts` and are covered by the bench assertions in §5.4.4; changing a constant without updating a bench expectation fails CI.
- The bench (`packages/bench`) runs synthetic cohorts over 90 simulated days in seconds (pure engine, no DB), produces trait and expression trajectories, and prints the assertion table. It is also the tool engineers use to answer "what happens if we change τ_p" before touching production.
- A second bench mode replays a randomly generated multi-device event stream through both the live path and the catch-up path and diffs the resulting state (equivalence).

---

## 6. Sync model

### 6.1 Canonical state and versions

One row set per aviary in Postgres is the aviary. `state_version` increments on every canonical write (tick, arrival, rename, settle, newcomer release). Snapshots carry it; clients keep the highest seen and ignore any older response (out-of-order delivery on flaky mobile networks). ETag/`If-None-Match` uses it so idle keepalives are `304`s.

### 6.2 Propagation

Polling, not push. Each visible client pulls at `next_pull_after_ms` (60 s by default, the tick cadence), plus immediately on: `visibilitychange` to visible; a render-frame gap > 5 s (suspend/resume detection using `requestAnimationFrame` timestamps against `performance.now()`); reconnect after a failed pull; and after every `events` batch that returned a plan. The server may lower `next_pull_after_ms` (to 15 s) while a transition of interest is in flight (arrival, chorus, relocation storm). WebSockets and SSE are deferred: 60 s snapshots of a few kilobytes are cheap, and every scenario the PRD describes is covered by the pull triggers above (ambiguity register A9).

Clock: the client computes `offset = server_time − local_receipt_time − rtt/2` per snapshot (median of last five) and schedules everything in server time. Two devices side by side hear the same call at the same moment.

### 6.3 Writes

Clients write only events. Each event carries a UUIDv7 idempotency key; the server deduplicates per aviary. Events are batched every 5 s or immediately for user-initiated events (offer, listen-in, settle). On `pagehide`, the remaining batch and a `session.end` go out via `navigator.sendBeacon`. Unsent events on hard crash are lost; the loss is bounded to ≤ 30 s of presence and is accepted (ambiguity register A10).

### 6.4 Multi-device semantics

| Situation | Behaviour |
|---|---|
| Laptop and phone open at once | Both pull the same snapshot; presence is the union of intervals (≤ 1× credit); each device renders its own listen-in mix |
| Listen-in on different birds on two devices | Both birds are credited; both intervals are real attention |
| Offer from both devices | Second within 45 s yields an empty plan; cooldowns are per bird per account |
| Settle on one device, other device active | `settled_at` set; the other device's next keepalive renders evening; any `initial`/`visible` pull or any interaction event from either device clears it (re-engagement) |
| Rename on one device | `state_version` bump; other device picks the name up within one pull; the bird id is unchanged so focus, captions, and narration continue uninterrupted |
| Sign-in on a new device mid-session | Same record; nothing to merge |
| Phone in a different timezone | `session.start` updates the account timezone; the next tick uses it; the palette on each device follows that device's local clock, the moods follow the account timezone. A one-off mismatch during travel is accepted |
| Revoked device session | Next pull is `401` with the matter-of-fact session message |

### 6.5 Why conflicts cannot happen

- No client submits absolute bird state, so there is nothing to overwrite. The API role cannot write `bird_traits` at all.
- The tick is serialized per aviary by an advisory lock and idempotent by `tick_index`, so two workers cannot double-apply a window.
- Events are consumed in `seq` order exactly once (`consumed_tick` stamped in the tick transaction).
- Presence union and per-account cooldowns make concurrent devices additive-but-bounded rather than conflicting.
- Client optimistic presentation (an offer prop, a settle ramp) is never sent back as state; the next snapshot reconciles by animation, never by a jump.

### 6.6 Failure handling

| Failure | Handling |
|---|---|
| Snapshot pull fails | Keep rendering from the last snapshot with the local call schedule; retry with backoff (2 s → 60 s); after 5 minutes of failures show the system panel copy for load failure above the scene, scene keeps moving |
| Event post fails | Retry with backoff, same idempotency keys; presence pings older than 10 minutes are dropped rather than replayed (they would be clipped by liveness anyway) |
| Catch-up over budget (> 2,000 ticks, sweep failure) | Run anyway, alert on-call, snapshot p99 alarm trips |
| Tick worker down | Warm aviaries fall behind; the next snapshot pull catches up inline, so users see continuity; alarm on scheduler lag > 3 min |
| Magic-link replay, session timeout, outage | System-voice surfaces from §4.8 |

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- Scene: one `<canvas>` (2D context), three offscreen layer canvases (background sky/foliage, midground perches and birds, foreground branch/leaf) composited each frame with a subtle parallax offset (≤ 6 px at 1080p) driven by a slow noise function, not by pointer position.
- Chrome: Preact for the top bar, panels (notebook, offer, settings), and the accessibility overlay. The scene has no framework dependency.
- Rendering resolution: `devicePixelRatio` capped at 2; the canvas is resized on viewport change with debounce, preserving aspect handling (§7.6).
- Animation clock: `requestAnimationFrame` with a fixed-step simulation of the behaviour state machine at 30 Hz and render interpolation at display rate; frame time budget in §10.

Why Canvas 2D: seven birds, a handful of ornaments, gradient backgrounds. Path2D drawing of ~30 paths per bird plus cached layer bitmaps costs ≈2–4 ms per frame on a 2019 integrated GPU laptop; WebGL would add context-loss handling and library weight for no visible gain. The renderer sits behind a `SceneRenderer` interface so a WebGL backend could be added if profiling on the baseline device ever disproves this.

### 7.2 First frame: the "already in motion" path

The first visible frame must be birds mid-action, not a load state, and it must happen under 500 ms on a mid-tier phone over 4G.

1. **Edge HTML assembly (cold load)**: the edge function validates the session cookie, fetches `/aviary/snapshot?reason=initial` from origin (origin p50 < 50 ms; edge-to-origin over a warm connection), and inlines the snapshot JSON plus a nonce'd inline script (`first-frame`, ≤ 15 KB gz) that: paints the sky field for the local hour, places each bird's silhouette and plumage at its perch slot in a species pose selected by mood, and starts a minimal breathing/scan animation. Total HTML ≈ 25 KB gz.
2. **Service worker (warm load)**: the app shell and the last snapshot are cached; on repeat navigation the shell paints from the cached snapshot immediately (first bird in < 100 ms) while the network snapshot arrives; reconciliation is by animation.
3. **Core takeover**: the core bundle (target ≤ 250 KB gz; hard budget §10) loads `defer`. On start it adopts the existing canvas, reads the first-frame clock, and continues the same animation without clearing. Poses used by `first-frame` are a build-time extract of the same pose library, so there is no visual pop.
4. **Audio and prose** load after first paint; nothing on the critical path waits for them.
5. **Budget arithmetic (4G, mid-tier Android, 4× CPU slowdown)**: edge TTFB ≈ 180 ms, HTML transfer ≈ 40 ms, inline parse + first paint ≈ 100 ms → first bird at ≈ 320 ms; margin ≈ 180 ms. Verified continuously by the synthetic fleet (§10.3).
6. When the snapshot is genuinely slow (origin degraded), the edge times out at 400 ms and serves the shell with a `pending` marker: the client paints the quiet field (soft sky gradient for the local hour with one slow light drift and, in continuous mode, one faint leaf) and fetches the snapshot itself. No spinner exists in the codebase.

### 7.3 Scene composition and the pose/motion system

- **Species art**: each species is a set of vector parts (body, head, wing, tail, crest) as compact path data, coloured from a plumage palette that is interpolated by `saturation_step` and layered by `detail_tier` (0: flat, 1: wing bars, 2: feather edges, 3: iridescent highlight pass). Compiled to `Path2D` at load; part bitmaps are cached per (species, pose, saturation_step, dpr) in a bounded LRU (≤ 40 entries) to bound memory.
- **Pose library**: per species ≈ 30 named poses (perch-neutral, fluffed, low-sleep, preen-wing-1..3, scan-left/right, head-tilt-l/r, step, hop-prep, hop-air, peck, drink, bathe, call-open, call-throat, alert-tall, wary-crouch). Poses are parameter sets over the part rig (rotations, offsets, scale), so interpolation between poses is a rig blend, not sprite swapping.
- **Behaviour state machine** (client, per bird): a mood-keyed distribution over micro-actions (idle-breathe, weight-shift, preen sequence, scan, tilt-toward-sound, fluff, hop-along-perch) with seeded timing from a per-bird noise stream (`hash(bird_id, day)`), so the same bird is recognisably itself and never loops. Sound events (any scheduled call, offer fragment) raise tilt probability for nearby birds for 3 s. Server-authored transitions (relocation, greeting plan, offer reaction, arrival) pre-empt the idle distribution.
- **Continuous renderer**: poses blend over 250–900 ms with eased curves and 1D noise on breathing amplitude (±2 %) and scan angle.
- **Flight**: relocations render as a two-segment curve (drop, glide, land) over the server-provided duration with a wing-cycle pose loop; landing triggers a weight-shift.
- **Ornaments**: leaves/feathers spawn client-side every 8–25 s (seeded per session), ≤ 3 alive at once, pooled objects (no allocation per spawn).
- **Day/night**: palette keyframes at 05, 07, 12, 17, 19, 21 local hours interpolated continuously; night dims the midground 45 % and adds a soft moon glow; the settle gesture blends to the 19:00 keyframe over 5 s and holds.
- **Weather**: rain as a pooled particle layer (≤ 120 particles) with a soft darkening of the sky; wind as a foliage sway amplitude increase and leaf spawn rate × 3. No thunder, no lightning, no snow.

### 7.4 Reduced-motion renderer

Selected by `prefers-reduced-motion: reduce` or the accessibility setting (`system | on | off`). Same behaviour state machine and same pose library, different renderer:

- Pose changes are cross-fades between fully rendered still poses, 1.5–3 s each, at most one change per bird per 6 s.
- Relocations: cross-fade out at the origin slot, cross-fade in at the destination (1.5 s each, overlapping).
- No breathing noise, no parallax, no leaves, no rain particles (rain becomes a slow sky darkening plus a still "wet perch" highlight).
- Day/night and settle transitions run at half speed.
- Calls, captions, narration, greetings, offers, and the notebook are unchanged.

Both renderers are exercised by the same visual regression suite (§11.4) so a feature landing in one cannot silently skip the other.

### 7.5 Reconciliation rules (snapshot → presentation)

- Perch differs: play a relocation of 1.2–2.4 s.
- Mood differs: finish the current micro-action (≤ 3 s), then draw from the new distribution; the plumage/posture register blends over 5 s.
- Expression step changed (drift crossed a boundary): blend palette/detail over 30 s. This is how "visible after three weeks" is not a snap in the fourth week.
- A bird is missing (released newcomer): fly-out over 3 s then remove.
- Call schedule: merge by seed; drop scheduled calls whose `at_ms` has passed by more than 2 s.

### 7.6 Responsive layout

The scene is composed in a normalized coordinate space (width 1000, height 460) with perch slots at fixed normalized positions. On a narrow viewport the scene scales uniformly to fit width and the middle/back zones compress vertically; on wide viewports perch spacing expands up to 1.4× while keeping all ten slots inside frame. Minimum supported viewport 320 × 480 (portrait phone): the three zones stack in depth rather than width, and all birds remain visible. No bird is ever culled or clipped.

### 7.7 Presence tracker (client)

A small module owns the three conditions: `document.visibilityState === 'visible'`, `document.hasFocus()`, and `lastActivity ≥ now − ACTIVITY_WINDOW` where activity is any `pointermove`, `pointerdown`, or `keydown` (pointer events include touch, so a tap on a phone counts; ambiguity register A6). `ACTIVITY_WINDOW` is 240 s initially, delivered by remote config so the beta can calibrate 180–420 s. While all three hold, it emits `presence.ping` every 30 s covering the interval since the last ping; on any condition failing it closes the interval at the last moment all three held (not at detection time), which also handles suspend/resume correctly. Rendering stops when the document is hidden (`requestAnimationFrame` stops naturally; the audio scheduler pauses, §8.6).

### 7.8 Chrome: top bar, panels, states

- Top bar: four icons (account/settings, accessibility settings, field notebook, offer) plus the settle gesture placed at the trailing edge. Fades to 8 % opacity after 4 s without pointer or keyboard activity; returns on activity. When keyboard focus is inside it, it never fades.
- Notebook panel: slides over the scene's edge without pausing the scene; entries virtualized (windowed list) so scroll-back over years does not retain DOM; weekday headers; no controls except close.
- Offer panel: three options with short naturalist labels; song fragments listed as five contour names.
- Settle: five-second undo means any pointer/keyboard interaction inside the scene reverses the ramp and sends `settle.undo`.
- Adoption surface (first run): shows the two arrived birds and name fields with suggestions, product voice, no exclamation marks.
- Loading and empty states: the quiet field (§7.2.6); after adoption, starters fly in on arrival plans.
- Settings, account, sessions, export, delete, visits: system voice, code-split, plain layout.

---

## 8. Audio pipeline

### 8.1 Architecture

One `AudioContext`, one `AudioWorkletNode` running a custom synth processor with nine persistent voices, connected to a small native-node mix: per-voice `GainNode` + `StereoPannerNode` (position from perch slot; front = wider, back = narrower and quieter) → bird bus → **listen-in mixer** (per-bird gain automation) → reverb send (feedback-delay-network reverb implemented inside the worklet, no impulse response file) → master `DynamicsCompressorNode` (soft-knee, as a safety limiter) → destination. No `AudioBuffer` is loaded from the network at any point; the build fails if an audio file appears in the bundle.

Why AudioWorklet with persistent voices: no per-call node creation, deterministic memory (zero allocation per call, satisfying the 30-minute no-growth rule), sample-accurate scheduling off the main thread, and the freedom to build bird-like timbres (FM plus filtered noise plus formant shaping) that native oscillators make awkward. All supported browsers ship AudioWorklet; there is no native-node fallback path (§8.7).

### 8.2 Call grammar runtime

A call is a structured object produced by `prose`-adjacent code in the `engine` package and consumed by both the synth and the caption generator:

```
Call {
  bird, phrase: contact|answer|alarm|chorus|night|greeting,
  motifs: [{ kind: rise|fall|arch|trill|buzz|whistle|chip, notes: [{pitch_st, dur_ms, amp}], gap_ms }],
  tempo, breathiness, pitch_center_st, spread_st
}
```

Expansion: species motif library (8–14 motifs each) → phrase grammar (probabilistic phrase structure: `contact := motif (gap motif){0,2}`, `answer := motif` echoing the caller's contour class, `alarm := chip{2,4}` short and sharp, `night := whistle` long and low, `greeting := rise (gap arch)?`) → bird signature → mood shaping → seeded variation of every duration, pitch, and amplitude by ±4–12 %. The same seed on any device yields the same call; a different seed always yields a different one.

### 8.3 Per-bird signature (recognizability)

Immutable from adoption, derived from `call_signature_seed`: base pitch offset within the species range (one of 7 semitone steps), a rhythm fingerprint (one of 6 inter-onset patterns applied to multi-note motifs), a timbre setting (one of 4 FM ratio/formant pairs), and a characteristic ornament (a grace note, a terminal buzz, none). Mood and vocal-frequency drift change *how often*, *how long*, and *how sharp*, never the four signature dimensions. The seven-bird recognizability ceiling is protected by construction: 7 × 6 × 4 × 3 signature combinations, and the arrival selector avoids assigning a signature that shares three of four dimensions with a resident bird.

Validation: a listening test (`packages/bench` tooling) where 12 participants learn seven birds over two sessions and then identify calls blind; target ≥ 80 % accuracy at seven birds, ≥ 90 % at four. Runs before beta and again before launch.

### 8.4 Mood and personality shaping

| Mood | Tempo | Pitch spread | Breathiness | Typical phrase |
|---|---|---|---|---|
| content | 1.0 | 1.0 | 0.3 | contact, occasional arch |
| curious | 1.05 | 1.2 (rising contours weighted) | 0.3 | contact with rise |
| alert | 1.3 | 0.7 | 0.1 | chip, short whistle |
| wary | 0.9, shorter motifs | 0.8 | 0.2 | single chip, long gaps |
| drowsy | 0.7 | 0.6 | 0.6 | soft single notes |
| resting | — | — | — | none (nightjar: night whistle) |

Perch zone shapes loudness and reverb send (front: −0 dB, dry; middle: −4 dB; back: −8 dB, more reverb). Boldness feeds the perch, so it reaches the ear indirectly and consistently with what the eye sees.

### 8.5 Mixing: chorus, listen-in, settle, weather, night

- **Chorus**: overlapping calls are offset so no two onsets coincide within 120 ms, and pitch centres are nudged ±15 cents apart when two birds share a pitch class, which avoids the beating/phase artefacts the PRD warns about. The compressor catches summed peaks. Chorus windows come from the server schedule so all devices agree.
- **Listen-in**: on engage, the focused bird's gain ramps to +3 dB and its reverb send to dry over 2.0 s (`setTargetAtTime`, τ = 0.6 s); all other birds ramp to −9 dB over 2.5 s. Floor is −12 dB: no bird is ever muted by listen-in. On disengage (click again, focus another bird, click empty space, keyboard focus leaves the scene, or presence ends), symmetric ramps. **Mix decay**: after 10 minutes of listen-in with no further interaction the emphasis relaxes halfway toward ambient over 60 s while the bird stays focused; the next interaction restores it. Ambiguity register A8.
- **Settle**: master gain −8 dB over 5 s; the engine's `settle_mult` reduces rate. Undo reverses within the 5-second window.
- **Weather**: rain is *not* rendered as audio (no rain loop, which would be recorded-style ambience by another name); the damping of calls and a slight extra reverb is the whole sonic weather.
- **Night**: rate zero except the nightjar; the reverb tail is lengthened 20 % at night so the single caller sits in a larger space.

### 8.6 Autoplay, focus, and lifecycle

Browsers block audible playback until a user activation. On load the client creates the context; if it is `suspended`, the scheduler still runs (so captions and narration describe real calls) and the context resumes on the first activation gesture anywhere in the page (a click on empty aviary space counts) with a 2-second fade-in. There is no "click to enable sound" banner: the top bar has exactly four icons and no speaker (ambiguity register A11). When the document is hidden the scheduler pauses and the context suspends after 30 s; on visible it resumes silently if activation was previously granted.

### 8.7 Fallback

If `AudioContext` or `AudioWorklet` is unavailable, or the context errors, the aviary plays in silence with captions on by default (`captions: auto` resolves to on). There is no recorded-audio fallback and never will be. The error is counted once in aggregate telemetry (`audio.context_unavailable`).

### 8.8 CPU and memory

Nine voices at 48 kHz with FM + noise + biquad per voice ≈ 1.5 % CPU per active voice on the baseline laptop; typical concurrency ≤ 3 voices. All voice state is preallocated typed arrays; the reverb uses fixed-size delay lines; scheduling messages are small structured clones with no retained references. The soak test (§11.5) measures worklet heap as well as main-thread heap.

---

## 9. Accessibility surfaces

Accessibility is a v1 deliverable with the same milestone gates as rendering and audio (§12), not a follow-up.

### 9.1 Screen-reader narration

- An `aria-live="polite"` region (visually hidden) whose single text node is *replaced* on each update, so queued announcements cannot pile up.
- Cadence at idle: one prose update every 30–60 s (uniformly random, seeded), generated client-side by `prose` from the current snapshot and recent local events (which birds are where, what they are doing, the light, the weather, the last call). Idle narration is skipped when nothing changed.
- Priority events (return-greeting, offer reaction, settle, arrival, newcomer naming) are narrated within one second and suppress idle narration for the next 20 s.
- Voice: identical grammar and lint as the notebook ("a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."). Bird names are used once the user has named them; unnamed newcomers are described.
- Pace setting in accessibility settings: slow (every 90–120 s) / normal / off. Narration pauses when the document is hidden.

### 9.2 Captions for calls

- Generated from the `Call` object at the moment the synth schedules it, so text and sound agree: contour kinds and counts map to vocabulary ("a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"). Position is described only when it distinguishes ("from the back perch").
- Rendered in the DOM overlay near the calling bird, fading in over 200 ms and out 800 ms after the call ends; ≤ 2 captions visible at once; `aria-hidden="true"` (narration covers screen readers, so nothing is announced twice).
- On by default when audio is unavailable or the user muted calls; otherwise off until enabled in accessibility settings.
- Contrast: text on a soft translucent scrim guaranteeing ≥ 4.5:1 against the brightest midday sky and the darkest night palette; the scrim opacity is derived from the current palette luminance so it stays as light as AA allows.

### 9.3 Reduced motion

Specified in §7.4. The setting surface offers `follow system / on / off`. The mode is honoured from the very first frame (the inline first-frame script reads the media query before drawing), so a reduced-motion user never sees a burst of motion before the preference applies.

### 9.4 Keyboard navigation and focus

- Tab order: top bar items (settings, accessibility, notebook, offer, settle) → the aviary scene (`role="group"`, `aria-roledescription="aviary"`, accessible name "the aviary") → panels when open.
- Entering the scene focuses the first bird (leftmost by current x). Arrow keys move between birds in x-order (roving `tabindex`); focus follows a bird's id when it relocates, not its old position. Enter engages listen-in on the focused bird; Enter again or Escape disengages; Escape from an unfocused-bird state leaves the scene to the top bar.
- Bird accessible name is prose, not a label: "pip — on the front perch, preening". Mood is conveyed by the activity words, never by a mood word.
- Offer panel: opened by its top-bar button or the shortcut `Alt+Shift+O`; fully arrow-key navigable; Escape closes. Settle from the top bar; the five-second undo also accepts any key press inside the scene.
- Focus indicator: a 3 px soft outline drawn by the overlay element with an outer 1 px dark halo and inner 1 px light halo so it reads at ≥ 3:1 against both bright day and dim night, following the design-system spec when it lands. The outline tracks the bird's rendered bounding box each frame.
- Chrome buttons are real `<button>`s with visible names on focus; the top bar does not fade while it holds focus.

### 9.5 Contrast

All user copy (top bar, panels, settings, system panels, captions, the visually rendered narration if a user enables it) passes WCAG AA. Automated checks run in CI against the palette extremes (05:00, 12:00, 21:00 keyframes, rain darkening, settled state) for every text style. The scene itself contains no user copy.

### 9.6 Voice split as code

`prose` exports two entry points with distinct TypeScript types: `ProductCopy` (lowercase, present tense, no second person, no exclamation) and `SystemCopy` (sentence case, direct). Components are typed to accept one or the other; a scene or notebook component cannot render `SystemCopy` and a settings or error panel cannot render `ProductCopy`. The lint for product copy rejects: capitalised sentence starts, `!`, "you"/"your" (except the naming form fields, which are allow-listed), "welcome", "streak", "visited", "days in a row", "level", "score", "achievement", and every trait name. The lint for system copy rejects lowercase sentence starts and bird-verb vocabulary. The accessibility settings surface is system voice by PRD rule; the narration and captions it configures are product voice.

### 9.7 Assistive-technology testing

Before beta: NVDA + Firefox, JAWS + Chrome, VoiceOver + Safari (macOS and iOS), TalkBack + Chrome; two sessions with screen-reader users and two with reduced-motion users from outside the team, repeated before launch. Exit criterion: participants can describe what a named bird is doing and can listen in, offer, and settle without sighted help, and describe the experience as calm rather than noisy.

---

## 10. Performance budgets and observability

### 10.1 Budgets and how each is met

| Budget (PRD) | Internal target | How it is met | Gate |
|---|---|---|---|
| Initial JS ≤ 2 MB gz | Critical path ≤ 300 KB gz (first-frame ≤ 15 KB, core ≤ 250 KB); audio ≤ 80 KB; prose ≤ 40 KB; species art ≤ 120 KB total | Manual chunks; procedural art and audio; settings/account/visits/notebook code-split; no framework in the scene | `size-limit` in CI, hard fail at 2 MB, warning above targets |
| First bird visible < 500 ms (mid-tier mobile, 4G) | p75 ≤ 350 ms cold, ≤ 120 ms warm | Edge-inlined snapshot + inline first-frame; service-worker last-known snapshot; nothing blocks on audio/prose | Synthetic fleet on throttled mobile profile every 10 min; alarm at p75 > 450 ms |
| 60 fps idle motion on a five-year-old laptop | Frame time p95 ≤ 8 ms, no long tasks > 50 ms during idle | Cached layer bitmaps, pooled ornaments, 30 Hz behaviour step with render interpolation, DPR cap 2 | Nightly 30-minute run on a baseline device profile; RUM frame-time histogram |
| No memory growth over 30 minutes | Heap delta ≤ 5 MB after forced GC; worklet heap flat | Preallocated voices, bounded bitmap LRU, pooled particles, virtualized notebook, no per-call allocation | Accelerated soak in CI (§11.5) and nightly real-time soak |
| Simulation-tick latency p99 alarm at 5 s | p50 ≤ 20 ms live tick, ≤ 25 µs per empty catch-up tick | Pure engine, one transaction per tick, indexed warm scan | p99 alarm at 5 s (PRD), warning at 500 ms |
| Snapshot latency | p95 ≤ 150 ms origin | Single query per table, expression cache, bounded catch-up | Alarm at p95 > 300 ms |

### 10.2 What we measure (aggregate-only RUM and synthetics)

Client → `POST /telemetry` with a fixed schema; the server rejects any payload with unknown fields (schema whitelist) and stores nothing per account:

- Page load and first-bird-visible timings (with coarse device class and country only)
- Render frame-time histogram buckets per session (no session id; buckets summed server-side)
- Audio: context state at first paint, resume latency, worklet errors, `audio.context_unavailable` count
- Snapshot pull success/latency, event post retries
- Session-duration histogram in 5-minute buckets (no account dimension, as the PRD allows)
- Server: request counts, latencies, error rates, tick latency, scheduler lag, catch-up tick counts per request, mail delivery outcomes

Synthetics: a Playwright fleet in four regions on desktop and throttled-mobile profiles runs the full session (load, greeting, listen-in, offer, settle) every 10 minutes against dedicated synthetic accounts, and a nightly 30-minute soak.

### 10.3 What we deliberately do not measure

- Anything keyed by account, bird, or email. No per-account session counts, no visit-frequency, no retention cohorts by interaction type, no offer-acceptance rates per bird, no drift distributions across accounts, not even anonymized. The PRD's privacy commitment forbids population-level analysis of how birds are interacted with, so the pipeline cannot express those queries: the telemetry store contains no such columns.
- Engagement funnels. There is no "activation" or "streak" metric anywhere in the dashboards.
- Visit counts across hosts (the visit log is per host, on demand; there is no global visits metric beyond request rates).

### 10.4 Privacy boundary as infrastructure

- `sim` database and telemetry store are separate systems; the telemetry service has no `sim` credentials and vice versa.
- OTel collector processor drops attributes matching UUID or email regexes before export; a CI test feeds synthetic spans with such attributes and asserts they are stripped.
- Application logs use a structured logger with a scrubber; the auth, invite, email-change, and export code paths have tests asserting no email-like string reaches log output. `account_id` (UUID) may appear at debug level for support.
- Support tooling reads a single account's state only in response to a user request, is audited, and never exports aggregates.
- The privacy policy page (system voice, plain text) names the aggregate categories and excludes per-bird interaction state, as the PRD requires.

### 10.5 Alarms and on-call

Tick p99 > 5 s; scheduler lag > 3 min; snapshot p95 > 300 ms; 5xx rate > 0.5 %; magic-link mail failure rate > 2 %; first-bird p75 > 450 ms on the mobile synthetic; audio worklet error rate > 1 % of sessions; catch-up ticks per request p99 > 1,600 (sweep falling behind); hard-delete job failure; export job failure.

---

## 11. Testing and quality gates

### 11.1 Engine unit and property tests
- Determinism: same inputs and seed → identical output, across Node and browser builds.
- Monotonic drift: for random event streams over 10,000 ticks, no trait ever decreases; zero drift with zero presence.
- Presence union: two sessions with overlapping intervals credit ≤ window length; unfocused-visible or hidden-focused intervals credit nothing (the client tracker tests cover each single-condition case).
- Mood: dwell respected; night resting for non-nightjar species; contagion bounded; no code path resets mood on session start (a grep-level test plus an integration test that opens a session and checks the row).
- Perch: distribution tests per boldness/mood cell; resting birds never relocate.

### 11.2 Equivalence and catch-up
Live-vs-catch-up property test over random multi-day event streams (§5.2); catch-up cost benchmark (≤ 25 µs per empty tick) as a CI performance assertion with a 2× tolerance.

### 11.3 Calibration bench
§5.4.4 cohorts and assertions run on every engine change. Bench output is attached to PRs that touch constants.

### 11.4 Client
- Visual regression (Playwright screenshots) for both renderers at three viewports and five palette keyframes, including the settled state, rain, and a seven-bird scene.
- Reconciliation tests: snapshot with a different perch/mood/expression step produces animation, never a frame with a discontinuity (pixel-diff between consecutive frames bounded).
- Accessibility CI: axe-core on every panel; keyboard e2e (tab into scene, arrow between birds, Enter/Escape listen-in, offer via shortcut, settle undo by key); live-region tests asserting narration cadence bounds and priority suppression; contrast checks at palette extremes.
- Voice lint on both copy catalogues and on every notebook/narration/caption template; a generated-corpus test renders 10,000 notebook entries and 10,000 captions and runs the same lint on the output.
- First-frame test: the inline renderer's frame and the core renderer's first frame differ by < 1 % pixels.
- Bundle size gates.

### 11.5 Soak (no memory growth)
CI: run the scene with fake timers at 20× speed for the equivalent of 30 minutes, cycling through calls, listen-in, offers, notebook open/scroll/close, weather, and a relocation storm; force GC via CDP; assert main-thread heap delta ≤ 5 MB and worklet heap flat. Nightly: the same in real time on the synthetic fleet.

### 11.6 Data and migrations
Every migration runs against a fixture database with 200 aviaries and asserts afterwards: same bird ids, same `species_id` and `call_signature_seed`, same trait values, same notebook text. Immutability triggers have tests. Hard-delete test asserts zero rows remain in any table for the account and the export object is gone.

### 11.7 API and security
Contract tests from the zod schemas; the snapshot denylist test (no numeric trait fields, no email fields); magic-link single-use and expiry; no account enumeration (identical responses and timings within tolerance); CSRF; rate limits; visitor session cannot reach any write endpoint or host-only field; revoked invite yields `410` on the next pull.

### 11.8 Audio
Golden-render tests: a fixed `Call` with a fixed seed renders to the same PCM (within float tolerance) on every build; a spectral test asserts per-bird signature dimensions are distinguishable; a chorus test asserts no two onsets within 120 ms and no unison pitch centres. Listening test with participants (§8.3) before beta and launch.

### 11.9 Load
k6 scenario: 50,000 warm aviaries ticking, 10,000 concurrent visible clients polling at 60 s plus event batches, 2,000 cold-aviary inline catch-ups per minute. Pass: budgets in §10.1 hold with headroom ≥ 2×.

---

## 12. Rollout

### 12.1 Milestones and exit criteria

| Milestone | Scope | Exit criteria |
|---|---|---|
| **M0 Foundations** (weeks 1–6) | Monorepo, protocol schemas, engine with bench, species data v0 (three species), pose library v0, synth prototype with signatures, first-frame path prototype, auth and account skeleton, privacy boundary infra | Bench assertions pass for *regular*, *abandoned*, *always-open*; live/catch-up equivalence passes; recognizability pilot ≥ 80 % at four birds with the team; first bird < 500 ms on a throttled-mobile lab profile from the edge prototype |
| **M1 Private alpha** (weeks 7–12) | Full session loop: greeting, presence, listen-in, offers, settle, notebook observer, day/night, weather, reduced-motion renderer, narration and captions v0, six species, visits flow behind a flag | 20 internal accounts for three weeks; no announcement UI anywhere (review); presence calibration study (activity window chosen); notebook rate within 1 per 2–4 days; a11y smoke with one external screen-reader user |
| **M2 Closed beta** (weeks 13–20) | Invite-only ~500 accounts in the `beta` environment; visits on; export and deletion; synthetic fleet live; RUM live; AT testing rounds; listening test at seven birds using accelerated arrivals in beta only | Three-week drift visible to beta participants in interviews without prompting numbers; recognizability ≥ 80 % at seven; all §10.1 gates green for two consecutive weeks; zero P1 a11y findings; voice review of every surface |
| **M3 Launch v1** (week 21–22) | Production with the fixed arrival schedule; visits off by default per account; kill switches ready | Launch checklist (§12.4) complete; on-call rota; privacy policy live |
| **M4 Post-launch calibration** (weeks 23–30) | Presence window and drift constants adjusted from beta learnings (never re-run on history); chorus tuning ahead of the first production cohort reaching three birds (~day 75) | Bench re-validated; listening test repeated on the production synth build |

### 12.2 Ramping birds per aviary
The engine, renderer, and mixer support seven from M1 (synthetic seven-bird scenes are in the visual and audio test suites), but production accounts reach a third bird only by age (~day 75), a fourth around day 150, and so on. The beta environment uses an accelerated schedule (days ÷ 10) so chorus quality at three to seven birds is validated before any production account gets there. The schedule is config; arrivals can be paused globally without any user-visible promise being broken, and resumed with the aviary-age clock unaffected. Production never accelerates.

### 12.3 Feature flags and kill switches
Visits (server-side kill switch; product default is off per account regardless); audio worklet path (fallback to silence+captions if a browser regression appears); newcomer arrivals (pause); notebook observer (pause writing, never delete); RUM ingestion; edge snapshot inlining (fallback to client fetch with the quiet field). Flags are evaluated per request, never mid-tick.

### 12.4 Launch checklist
Budgets green two weeks running; AT sessions passed; voice lint green and manual voice review of all surfaces; snapshot denylist test green; privacy boundary tests green; backup/restore rehearsal with migration-invariant test; magic-link deliverability check across major mail providers; unsupported-browser page verified; export and hard-delete rehearsed end to end; on-call runbooks for tick lag, catch-up storms, mail failures, and worklet regressions.

### 12.5 Instrumented from day one
Everything in §10.2, plus: engine version per tick, catch-up tick counts, notebook write counts per day (aggregate), arrival counts per day (aggregate), visit flow error rates, and bench results attached to each engine release. Nothing per account.

### 12.6 Beta research and consent
Production never analyzes bird or interaction data. Beta participants sign an explicit research consent that is separate from the product privacy policy; only the `beta` environment may be inspected for calibration, only by named engineers, only for consented accounts, and the environment is destroyed at launch. This is the sole way drift calibration is validated on real behaviour.

---

## 13. Risks

| Risk | What goes wrong | Mitigation | Detection |
|---|---|---|---|
| Drift too fast or too slow | Birds visibly change between sessions (Tamagotchi feel) or never seem to change (screensaver feel) | Bench cohorts with hard assertions; saturation curve; expression steps sized to JNDs; beta interviews at three weeks | Bench in CI; beta interviews; M4 recalibration window |
| Presence inflation | Laxer-than-spec presence (background tabs, always-open laptops) corrupts drift for everyone | Three-condition tracker with unit tests per condition; server liveness clipping; union across devices; *always-open* bench cohort must yield zero | Bench; audit of presence code paths in review |
| Presence under-credit on touch devices | Phone users watching without touching lose presence after the window | Pointer events include taps; longer window on coarse pointers considered in the alpha study | Alpha calibration study |
| Sync correctness | Double-credited presence, lost drift, or divergence between devices | No client state writes; advisory-locked idempotent ticks; event `seq` consumption; equivalence test; multi-device e2e | Property tests; the e2e suite; snapshot version monotonicity assertions in the client |
| Catch-up storms | Sweep falls behind, inline catch-up balloons, snapshot latency blows the first-bird budget | Daily sweep with bucketing; bounded inline (≤ 1,440); alarm on catch-up counts; edge timeout to quiet field | Catch-up p99 alarm; snapshot p95 alarm |
| Audio uncanniness | Synth sounds like a synthesizer; chorus turns to mush; phase artefacts; sameness | FM+noise+formant voices designed with an audio designer; onset spreading and detune; signature dimensions; calm cap; listening tests at four and seven birds | Listening tests before beta and launch; beta feedback |
| Autoplay policy | First session is silent until a gesture and the user never learns there is sound | Scheduler runs regardless; resume on first activation with fade-in; captions describe calls when silent | Aggregate `audio.resume_latency`; beta interviews |
| Accessibility regression | A feature lands in the continuous renderer only, narration floods, focus lost on relocation, contrast fails at dusk | Both renderers in the same visual suite; live-region cadence tests; focus-follows-id tests; palette-extreme contrast CI; external AT rounds | CI gates; AT sessions |
| Voice leakage | A toast, a "welcome", a capitalised notebook entry, or system tone in the scene | Typed copy catalogues; no notification component; voice lint on templates and generated corpora; manual voice review at each milestone | Lint; review |
| Gamification creep | A "harmless" counter or calendar appears | Non-goals restated as engineering consequences (§1.2); no per-day presence data outside the tick; review checklist | Review; schema denylist |
| PII leakage | Email in logs, metrics, or partition keys | Single encrypted column; blind index; scrubber tests; OTel attribute filter; UUID everywhere | CI tests; log audits |
| First-bird budget on cold loads | Edge-to-origin latency in far regions blows 500 ms | Regional origin replicas or edge KV cache of the last snapshot per account (private, 60 s TTL) as a fallback; quiet field beyond 400 ms | Synthetic fleet per region |
| Newcomer flow tone | Arrival reads as an announcement or a reward | Arrival is silent, on the back perch, notebook-only; naming is optional and reachable; release is quiet | Voice review; beta interviews |
| Magic-link deliverability | Users cannot sign in | Reputable transactional provider, DKIM/SPF/DMARC, provider failover, deliverability checks | Mail failure alarm |
| Species art scope | Six species with 30 poses each and four detail tiers is a large art task | Start with three species through M1; procedural detail tiers derived from base art; art pipeline tooling early | Milestone tracking |
| Calibration blind spot in production | Privacy forbids population analysis, so real-world drift cannot be observed after launch | Beta consent environment; bench; qualitative user feedback channel; conservative constants | Support feedback trends (qualitative) |

---

## 14. Ambiguity register (decisions made without asking)

| # | Ambiguity | Decision |
|---|---|---|
| A1 | "The tick runs whether or not any client is connected" vs. cost of ticking every account every minute | Warm aviaries tick live; cold aviaries advance by deterministic catch-up that is bit-identical to live ticks (pure engine, seeded RNG). The observable property the PRD wants holds; the daily sweep bounds catch-up |
| A2 | Whether the snapshot may contain the personality vector | No. Clients receive quantized expression profiles; the vector leaves the server only in the account export, which the PRD explicitly specifies |
| A3 | How "a new species offer appears in the user's flow" | A newcomer arrives quietly on the back perch as a full unnamed bird; naming (adoption) and "let it move on" (release) live in the bird's focus panel and settings; no prompt or notification; the notebook records it |
| A4 | Bird sleep state is called "settled" in `aviary_layout.md`, but `concepts.md` reserves "settle" for the gesture and lighting | Internal mood is `resting`; prose may still say "settled low on the perch" |
| A5 | Offer targeting and cooldown UI | Offers are aviary-wide; every eligible bird reacts by mood and curiosity; per-bird cooldown of 5 min after a reaction; aviary debounce 45 s; one prop at a time; no cooldown indicator |
| A6 | "pointermove or keypress" on touch devices; activity window length | Any pointer event (move, down) or keydown counts; window 240 s initially, calibrated 180–420 s in alpha |
| A7 | The brief lists muting calls among drift-shaping behaviours; the engine file does not | Presence while muted drifts everything, with the vocal-frequency component at half rate; never negative |
| A8 | "Listen-in mix decay" | Slow ramps on engage/disengage plus a relaxation of emphasis after 10 minutes of inactivity while focus persists; auto-disengage when presence ends |
| A9 | Push vs pull for multi-device propagation | Pull at tick cadence with visibility, frame-gap, and post-event triggers; no WebSockets in v1 |
| A10 | Event loss on hard tab kill | Accept ≤ 30 s of lost presence; `sendBeacon` on `pagehide`; no client-side persistence of events |
| A11 | Browser autoplay blocking vs "calls already audible" on first frame | Scheduler runs from first frame; audible on first activation gesture with a 2 s fade; no speaker icon (top bar has exactly the four PRD icons plus settle) |
| A12 | Is "settled" per device or per account? | Account-level `settled_at`; cleared by any re-engagement (initial/visible pull or interaction) from any device |
| A13 | Does a greeter always exist? Do visitors get greeted? | A greeter always exists (nightjar or minimal greeting at night); visitors never receive a greeting plan because the birds notice the host, not the visitor |
| A14 | How much to trust client presence claims | Credited only if the session pulled a snapshot within 180 s; clipped to session bounds; unioned per account |
| A15 | "Regular visits" for calibration | Five 15-minute sessions per week with one listen-in each |
| A16 | Multiple devices in different timezones | Last-reported timezone wins for the engine; palette follows each device's clock |
| A17 | Export contains vectors | Yes, per PRD; it is the only exposure and is not a UI surface |
| A18 | "One-time link" for visits vs "active invite" | The link is single-use; consuming it creates a visitor session that lasts until revocation, invite expiry, or 30 days of inactivity |
| A19 | Engine upgrades during catch-up | Missing spans are computed with the current engine; history is never re-run; accepted as unobservable |
| A20 | What a "song fragment from a small library" is | Five procedural contour motifs (rise, fall, arch, level, two-part) in a hummed timbre; never recorded audio |
| A21 | Validating calibration in production under the privacy rule | Never in production; consented beta environment only (§12.6) |
| A22 | Top-bar keyboard shortcut for offers | `Alt+Shift+O` (modifier combo to satisfy WCAG 2.1.4), listed in accessibility settings |
| A23 | Weather at night | None scheduled 00–05 local; it would only dampen the one night caller and no one would see it |

---

## 15. Workstreams and sequencing

| Workstream | People | Owns | Depends on |
|---|---|---|---|
| Engine | 2 engineers | `engine`, `bench`, tick worker, calibration, observer detectors | Protocol schemas (week 1) |
| Platform | 2 engineers | `api`, auth, sessions, export/delete, visits, edge function, privacy infra, telemetry pipeline, load tests | Protocol schemas |
| Scene | 2 engineers + visual designer | `client` renderer (both modes), first-frame path, species art pipeline, pose library, chrome, presence tracker | Protocol; species art |
| Audio | 1 engineer + audio designer | worklet synth, grammar runtime, signatures, mixing, listening tests | Engine call schedule format (week 2) |
| Prose and accessibility | 1 engineer + writer | `prose`, templates, voice lint, narration, captions, keyboard/focus, AT testing, copy catalogues | Engine state shapes; overlay from Scene |

Sequencing: week 1 freezes the protocol schemas (snapshot, events, plans, `Call`), which unblocks all five workstreams in parallel. The engine bench and the first-frame prototype are the two earliest de-risking artefacts (M0). Species art is the long pole; three species are enough through M1, six by M2. AT rounds are scheduled twice (M2 entry and exit), not once at the end.

---

## Appendix A. Initial constants (all remote-config or engine-versioned; values are starting points for calibration)

| Constant | Value |
|---|---|
| Tick interval | 60 s |
| Warm window after last snapshot pull | 10 min |
| Inline catch-up bound | 1,440 ticks; alarm above 1,600 |
| Presence ping interval / max interval | 30 s / 35 s |
| Presence liveness requirement | snapshot pull within 180 s |
| Activity window | 240 s (calibrate 180–420) |
| Presence daily saturation τ_p | 90 min |
| Listen-in daily saturation τ_l | 20 min per bird |
| Presence coefficients (per day at full credit) | boldness 0.030, warmth 0.025, vocal 0.030 (×0.5 muted), plumage 0.045, curiosity 0.020 |
| Listen-in coefficients | warmth 0.020, vocal 0.020 |
| Offer accepted / near | curiosity +0.004, boldness +0.002 per event, ≤ 3 per bird per day |
| Saturation exponent | (1 − t)^1.5 |
| Mood dwell | 8–40 min by mood; softmax temperature 0.35 |
| Relocation hazard | 1 per 20 min baseline |
| Weather | rain 2.5/week 3–8 min; wind 4/week 2–5 min; none 00–05 local |
| Spontaneous call rate | (0.3 + 1.2·vocal) per min × multipliers; refractory 6 s; aviary cap 6 per min |
| Response probability | (0.15 + 0.5·warmth) × mood multiplier, ≤ 1 responder per call |
| Chorus | trigger 2 birds vocal ≥ 0.5 within 10 s; window 20–40 s |
| Greeting start | 600–1,500 ms after first frame; absence threshold 30 s; responder stagger 1.5–6 s; min 800 ms between actions |
| Offer | debounce 45 s; per-bird cooldown 5 min; prop lifetime seed 3 min, pool 3 min, song 20 s |
| Settle | ramp 5 s; undo window 5 s; master −8 dB; rate × 0.3 |
| Listen-in mix | focused +3 dB dry; others −9 dB (floor −12 dB); ramps 2.0/2.5 s; relaxation after 10 min |
| Notebook governor | capacity 2.0; refill 1 per 72 h (active) / 168 h (absent); thresholds 0.35–0.7; overdraw for s ≥ 0.85 |
| Newcomer schedule (days) | 75, 150, 240, 330, 450 (± 15 %) |
| Magic link | 15 min expiry; 5/h and 20/day per address; 30/h per IP |
| Invite expiry | 30 days; visitor session 30 days sliding |
| Soft delete window | 30 days; backups 14 days |
| Event log retention | 30 days after consumption; presence bookkeeping 3 days |
| Bundle | critical ≤ 300 KB gz target, 2 MB hard; first-frame ≤ 15 KB |
| Frame budget | p95 ≤ 8 ms on baseline laptop; DPR cap 2 |
| Soak | heap delta ≤ 5 MB over 30 min |

## Appendix B. Internal names vs PRD vocabulary

| Internal | PRD language | Note |
|---|---|---|
| `resting` mood | "settled (eyes closed, low on the perch)" in `aviary_layout.md` | Avoids collision with the settle gesture and settled lighting state |
| expression profile | (none; derived from the personality vector) | Quantized presentation parameters; never numbers in the UI |
| responder | (none) | Request-path planner for greetings and offer reactions; writes no bird state |
| catch-up tick | "the tick runs whether or not any client is connected" | Deterministically equivalent to live ticks |
| newcomer | "a new species offer appears" | Unnamed full bird until adopted (named) or released |
| quiet field | "the loading state is a quiet field" | Also the empty-aviary state |
| `ProductCopy` / `SystemCopy` | naturalist voice / matter-of-fact voice | Enforced by types and lint |
| plan | (none) | Server-authored greeting, offer-reaction, or arrival sequence executed by the client |
