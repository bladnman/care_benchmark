# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. Mechanical search of the frozen reconstruction found no gold IDs or external score taxonomy tokens such as `F1`, `S1`, `R-F01`, or `R-S01`. The reconstruction uses plan-native sectioning and feature names rather than gold identifiers.

## Vocabulary Check

Checked scorer-side terms: `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, and `intent fidelity`. None appeared in `RECONSTRUCTION.md`.

Sampled plan-derived phrases from the reconstruction:

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| "calm, already-in-progress place" | PLAN §1 says the browser renders a "calm, already-in-progress place". | plan-derived |
| "server is the only writer" | PLAN §1 invariant: "The server is the only writer". | plan-derived |
| "visible document + window focus + recent pointer or key activity" | PLAN §1 invariant and §4.2 presence endpoint use the same conjunction. | plan-derived |
| "matter-of-fact system language" | PLAN §1 voice invariant names matter-of-fact system language. | plan-derived |
| "aggregate operational metrics only" | PLAN §2.1 telemetry gateway accepts aggregate operational metrics only. | plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required by the reconstructor prompt for missing rationale. | expected |

## Heading Mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then `### Delivery intent and scope`, `### Proposed architecture`, `### Data model`, `### API surface`, `### Simulation engine`, `### Frontend rendering pipeline`, `### Audio pipeline`, `### Accessibility and content surfaces`, `### Performance, observability, and privacy controls`, `### Rollout and operational plan`, `### Risks and mitigations`, and `### Definition of done`.

The two top headings are mandated by the phase-2A prompt. The `###` headings mirror the PLAN sections, not the held-out gold-list sections. No heading mirrors S1-S9/F1-F40 labels or gold-list group titles closely enough to suggest contamination.

## 1:1 Mapping Suspect

No 1:1 mapping to the gold target list was found. The reconstruction does not enumerate S1-S9 or F1-F40, does not keep the gold order, and includes many plan-native items beyond the 49 scored whys. Its per-feature section follows the candidate PLAN's implementation sections and includes numerous `NOT RECOVERABLE FROM PLAN` judgments, which is consistent with blind plan-derived reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Finding |
|---|---|---|
| "Build a calm, already-in-progress place rather than an app that begins from a loading or static state." | PLAN §1: web-only aviary where the browser renders "a calm, already-in-progress place"; PLAN §6.1 forbids spinner/fade-from-static. | supported |
| "Keep canonical life on the server and keep clients as renderers and event submitters." | PLAN §1 invariants: server is only writer; clients submit append-only facts. PLAN §2.2 says client rendering can pause while server tick never pauses. | supported |
| "Protect privacy through synthetic identifiers, minimum telemetry, and separation from analytics." | PLAN §2.1 says internal references use a generated account UUID and the telemetry gateway has no read access to the simulation database; PLAN §9 removes account/bird dimensions from telemetry. | supported |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold ID leakage, no scorer/rubric vocabulary, no suspicious gold-order mapping, and its headings mirror the assigned PLAN rather than the gold list. The few strong phrases checked above have direct support in the PLAN.
