# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction found no gold IDs or scorer taxonomy tokens such as `F1`, `S1`, `R-F01`, `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`. The reconstruction uses plan-native section names and feature descriptions, not the held-out gold ID scheme.

## Vocabulary Check

Sampled phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-owned canonical state over client authority" | PLAN says "clients never advance simulation" and "the server is the sole writer of canonical vectors". | Plan-derived. |
| "A calm naturalist aviary, not a game or social network" | PLAN requires naturalist copy and excludes leaderboards, feeds, comments, chat, public discovery, gamification, and caretaker mechanics. | Plan-derived. |
| "Non-punitive care and absence" | PLAN says "birds never punish absence" and no trait moves downward because of neglect. | Plan-derived. |
| "Slow, monotonic, hidden personality drift" | PLAN says drift is slow/monotonic and personality values are not exposed in product UI. | Plan-derived. |
| "Privacy and minimization as structural boundaries" | PLAN uses synthetic UUIDs, encrypted email-only account records, aggregate-only telemetry, and analytics isolation. | Plan-derived. |
| "Accessibility as a launch contract" | PLAN says accessibility features ship at launch and are release criteria. | Plan-derived. |
| "Already-active, continuous one-screen presence" | PLAN says the scene appears already in motion and avoids spinner/static wake-up. | Plan-derived. |
| "Deterministic, idempotent correctness" | PLAN emphasizes idempotency keys, retry-safe ticks, deterministic fixtures, and replay tests. | Plan-derived. |
| "Opt-in, read-only, simulation-neutral visits" | PLAN states visits are opt-in, read-only, revocable, expiring, and cannot influence host state. | Plan-derived. |

I found no rubric-side vocabulary used freely in the reconstruction.

## Heading Mirror

The only markdown headings in the reconstruction are:

| Heading | Comparison | Assessment |
|---|---|---|
| `## System-level intent` | Required by the phase-2A prompt, not a gold-list title. | Not suspicious. |
| `## Per-feature whys` | Required by the phase-2A prompt, not a gold-list title. | Not suspicious. |

The reconstruction uses bold plan-section/group labels under the required headings. These mirror PLAN.md sections, not GOLD_WHYS.md titles.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40, does not follow the gold list order, and includes many more plan-derived feature bullets than the 40 scored feature whys. Its organization follows the PLAN structure: scope, architecture, data model, API, simulation, frontend, audio, accessibility, privacy/security, performance, rollout, and risks.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The server owns account identity, bird identities, names, species, personality vectors, moods, simulation timestamps, pending transitions, interaction event ordering, notebook entries, and invitation/revocation status." | PLAN section 2 states the same ownership list nearly verbatim. | Supported. |
| "Presence collection requires the conjunction of visible document, focused window, and pointer movement or keypress within a calibrated few-minute activity window." | PLAN section 5 uses the same three-signal conjunction and calibration language. | Supported. |
| "If WebAudio is missing or denied, use graceful silence and enable captions by default; do not ship recorded fallback audio." | PLAN section 7 states the same fallback rule. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no held-out IDs, no scorer vocabulary, no gold-list ordering, and its articulate claims are traceable to PLAN.md. The main scoring issue is compression of rationale into mechanisms, not contamination.
