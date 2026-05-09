# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no ID leakage found.

I searched the frozen reconstruction for gold IDs and rubric-style identifiers such as S1-S9, F1-F40, R-Fxx, weight-3, multi-layer, feature-level fidelity, and intent fidelity. The only notable loaded phrase was "load-bearing," which also appears in the PLAN and is therefore plan-derived rather than leakage.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "architectural absences" | yes | PLAN Scope uses this exact framing. |
| "The client never owns canonical anything" | yes | Exact PLAN architecture sentence. |
| "load-bearing line" | yes | PLAN 2.2 uses this phrase. |
| "Privacy is architecture, not an addendum" | yes | PLAN Privacy section says this directly. |
| "Sparsity is a feature, not a bug" | yes | PLAN notebook generation uses this exact sentence. |
| "not a degraded fallback" | yes | PLAN reduced-motion section uses this phrasing. |
| "first frame is alive" | yes | PLAN out-of-band notes use this promise. |
| "NOT RECOVERABLE FROM PLAN" | format-derived | Expected phase-2A reconstruction marker, not gold content. |

No rubric-side vocabulary such as "intent fidelity," "feature-level fidelity," "multi-layer recovery," or "weight-3" appears in the reconstruction.

## Heading Mirror

Reconstruction headings are plan-shaped, not gold-shaped:

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| "System-level intent" | phase-2A output format | Expected structural heading. |
| "Per-feature whys" | phase-2A output format | Expected structural heading. |
| "Scope" | PLAN section 1 | Plan mirror, not gold mirror. |
| "Architecture" | PLAN section 2 | Plan mirror, not gold mirror. |
| "Simulation Engine" | PLAN section 5 | Plan mirror, not gold mirror. |
| "Sync Model" | PLAN section 6 | Plan mirror, not gold mirror. |
| "Frontend Rendering Pipeline" | PLAN section 7 | Plan mirror, not gold mirror. |
| "Audio Pipeline" | PLAN section 8 | Plan mirror, not gold mirror. |
| "Accessibility Surfaces" | PLAN section 9 | Plan mirror, not gold mirror. |
| "Performance Budgets and Observability" | PLAN section 10 | Plan mirror, not gold mirror. |
| "Privacy / Telemetry Boundary" | PLAN section 11 | Plan mirror, not gold mirror. |

No heading sequence mirrors GOLD_WHYS section titles or S/F ordering.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not walk S1-S9 or F1-F40, does not preserve gold IDs, and does not assign a neat item to every gold target. It is organized by the PLAN's own sections and includes many non-gold implementation items, such as service shape, endpoint details, render pipeline choices, audio node pooling, and rollout sequencing.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan says the out-of-scope items are 'not just deferred features' but 'architectural absences'."
   PLAN support: Scope states the out-of-scope items are "not just deferred features" and "architectural absences."

2. Reconstruction: "The server owns the aviary; the client renders it."
   PLAN support: Architecture says the server owns personality, mood, simulation time, notebook, accounts, sessions, invitations, and the event log; client owns rendering/audio/input state.

3. Reconstruction: "Reduced-motion mode is 'not a degraded fallback' but 'the same product, slower'."
   PLAN support: Frontend reduced-motion section says the mode is not a degraded fallback and describes it as the same product, slower.

All three articulate sentences are directly derivable from PLAN passages.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It contains no gold IDs, no rubric scoring vocabulary, no gold-order mapping, and its strongest phrases trace back to the PLAN. The few phase-specific phrases are expected reconstruction-format markers rather than evidence of gold-list contamination.
