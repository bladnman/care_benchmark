# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: PASS. A targeted search for gold/rubric identifiers and terms (`S1`, `F1`, `R-F01`, `weight-3`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `gold`, `rubric`) found no hits in `RECONSTRUCTION.md`. The reconstruction does not use gold IDs or external rubric taxonomy.

## Vocabulary Check

Sampled reconstruction phrases against `PLAN.md`:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "quiet, browser-based virtual aviary" | PLAN.md:5 exact phrase | plan-derived |
| "long-term, low-key observational companionship" | PLAN.md:5 exact phrase | plan-derived |
| "Feels alive, not robotic" | PLAN.md:9 exact principle | plan-derived |
| "Notice, never announce" | PLAN.md:10 exact principle | plan-derived |
| "naturalist, lowercase, present-tense observations" | PLAN.md:11 exact phrase | plan-derived |
| "server-driven" and "deterministic renderers" | PLAN.md:34-35 matching architecture language | plan-derived |
| "Last-Write-Wins hazards" | PLAN.md:441 section wording | plan-derived |
| "STRICT ISOLATION" | PLAN.md:612 exact phrase | plan-derived |
| "charm-preserving surface rather than a compliance checklist" | PLAN.md:566 near-exact phrase | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Not a PLAN phrase; it is phase-2A self-audit language | acceptable, not gold-specific |

No rubric-side vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," or gold IDs appears in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Executive Summary & Scope`
- `### 2. System Architecture & Service Boundaries`
- `### 3. Data Model & Database Schemas`
- `### 4. API Surface & Protocols`
- `### 5. Simulation Engine & Mathematical Models`
- `### 6. Multi-Device Sync & Conflict Model`
- `### 7. Frontend Rendering Pipeline`
- `### 8. WebAudio Procedural Synthesis Pipeline`
- `### 9. Accessibility Surfaces`
- `### 10. Performance Budgets, Telemetry & Privacy`
- `### 11. Rollout & Population Ramp Plan`
- `### 12. Engineering Risks & Mitigations`

The `###` headings mirror the PLAN section headings and order, not the gold-list sections. `## System-level intent` and `## Per-feature whys` match the expected reconstruction contract, not a gold taxonomy leak. No near-exact gold section titles such as "System-level whys" or "Feature-level whys" appear.

## 1:1 Mapping Suspect Check

Verdict for this check: PASS. The reconstruction does not enumerate S1-S9 or F1-F40, does not produce 49 neat rows, and does not follow the gold order. It follows the PLAN's 12-section implementation structure and includes many plan-local features that are not gold-why rows, such as Redis, PostgreSQL, WebAudio node pooling, and implementation phases. Several rows are explicitly marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than gold-list mirroring.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "This intent recurs in the v1 scope forbidding streaks, levels, XP, badges, scores, health bars, mortality, neglect penalties, and other game or custody mechanics." | PLAN.md:23-24 forbids streaks, levels, XP, badges, scores, mortality, neglect penalties, etc. | grounded |
| "Because the server simulation tick is the sole entity calculating and applying additive deltas, concurrent sessions on multiple devices cannot overwrite or erase accumulated drift history." | PLAN.md:441-443 says the server tick is sole calculator/applicator of additive deltas and concurrent sessions cannot overwrite drift. | grounded |
| "The plan permits request rates, status codes, latencies, tick runtimes, FPS percentiles, and WebAudio errors while forbidding user IDs, synthetic IDs, bird names, traits, moods, interaction frequencies, and per-user session durations." | PLAN.md:614-621 lists the allowed and forbidden telemetry fields. | grounded |

## Verdict

PASS. The reconstruction reads as plan-derived: it mirrors PLAN headings, reuses PLAN vocabulary, omits gold IDs/rubric terms, and lacks a suspicious 1:1 map to the gold why list. The only non-plan vocabulary of note is the explicit `NOT RECOVERABLE FROM PLAN` marker, which is appropriate phase-2A self-audit language rather than evidence of gold contamination.
