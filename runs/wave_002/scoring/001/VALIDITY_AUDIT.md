# VALIDITY_AUDIT - CARE run 001

## ID leakage

Verdict: PASS. I found no gold IDs such as S1-S9, F1-F40, R-F identifiers, or gold/rubric taxonomy in the frozen reconstruction. A regex check produced only the ordinary word "Low-fidelity" as a false positive for an F-like pattern; the same phrase appears verbatim in PLAN.md.

## Vocabulary check

| Reconstruction phrase | Present in PLAN? | Assessment |
|---|---|---|
| "low-fidelity, high-affect" | yes, PLAN Scope | Plan-derived. |
| "continues without the viewer" | yes, PLAN Architecture | Plan-derived. |
| "Canonical Server / Interpolating Client" | yes, PLAN Architecture | Plan-derived. |
| "presence-time" | yes, PLAN Simulation | Plan-derived. |
| "server is the single source of truth" | yes, PLAN Sync Model | Plan-derived. |
| "No audio loops" | yes, PLAN Audio Pipeline | Plan-derived. |
| "Naturalist Voice" rubric | yes, PLAN Risks | Plan-derived. |
| "last-write-wins" | yes, PLAN Sync Model | Plan-derived. |
| "Per-feature whys" | no | Scoring/reconstruction scaffold wording, not a gold-list mirror. Minor but expected. |
| "NOT RECOVERABLE FROM PLAN" | no | Phase-2A methodology phrase, not contamination by gold content. |

I did not find loaded rubric-side phrases such as "multi-layer recovery", "feature-level fidelity", "weight-3", "gold why", or "intent fidelity" in RECONSTRUCTION.md.

## Heading mirror

RECONSTRUCTION.md has two top-level headings, "System-level intent" and "Per-feature whys", then mirrors the PLAN's numbered section headings: Scope, Architecture, Data Model, API Surface, Simulation Engine Design, Sync Model, Frontend Rendering Pipeline, Audio Pipeline, Accessibility Surfaces, Performance and Observability, Rollout, and Risks and Mitigations. This is a mirror of PLAN.md structure, not GOLD_WHYS.md structure. It does not enumerate S1-S9 or F1-F40.

## 1:1 mapping suspect

Verdict: PASS. The reconstruction does not give every gold target a neat corresponding item in gold order. It walks PLAN sections and plan bullets, including many non-gold items and many "NOT RECOVERABLE FROM PLAN" entries. The ordering and grouping match PLAN.md, not the gold list.

## Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The plan gives the server the only writer role for personality vectors, while the client submits append-only events." | PLAN Client/Server Split says the server owns all state and is the only writer; Event API is append-only. | Supported. |
| "Screen-reader narration is explicitly descriptive prose instead of state labels." | PLAN Screen-Reader Narration says naturalist descriptive prose instead of state labels. | Supported. |
| "Additive changes are applied in event log order to prevent last-write-wins data loss across devices." | PLAN Sync Model says additive deltas are based on event log order and prevent last-write-wins data loss. | Supported. |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no gold-order mapping, no suspicious rubric vocabulary, and its headings mirror the plan rather than the gold list. The only non-plan vocabulary is ordinary phase-2A scaffold language ("Per-feature whys" and "NOT RECOVERABLE FROM PLAN"), which is not enough to suggest contamination.
