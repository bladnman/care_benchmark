# VALIDITY_AUDIT - CARE run 001

## ID leakage check

Verdict: no ID leakage found.

I searched the frozen reconstruction for gold-side identifiers and rubric terms including S1-S9, F1-F40, R-Fxx, gold, rubric, weight-3, feature-level fidelity, intent fidelity, multi-layer, and load-bearing. There were no hits in RECONSTRUCTION.md and no matching hits in PLAN.md. The reconstruction does not use gold IDs or the scoring taxonomy.

## Vocabulary check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "no streaks, levels, scores, badges, achievements" | yes | Directly plan-derived from strict non-goals. |
| "alive over weeks" | yes | Directly plan-derived from Risks. |
| "low-pass filter over presence and interaction signals" | yes | Directly plan-derived from Simulation engine design. |
| "append-only event log" | yes | Directly plan-derived from Data model and Architecture. |
| "naturalist prose" | yes | Directly plan-derived from Accessibility surfaces. |
| "cheap ARIA labels" | yes | Directly plan-derived from Risks. |
| "phase-canceling in chorus" | yes | Directly plan-derived from Audio pipeline. |
| "bounded, calm social presence" | no exact phrase | A synthesized summary of opt-in/read-only/no-social-network rules, not a gold/rubric phrase. |
| "Privacy-conscious instrumentation" | no exact phrase | A synthesized heading from synthetic UUID and privacy-boundary telemetry plan content. |
| "first-class accessibility surfaces" | close | PLAN says accessibility is in V1 scope and calls them Accessibility surfaces; not rubric-only. |

No suspect rubric vocabulary such as multi-layer recovery, feature-level fidelity, weight-3, load-bearing, or intent fidelity appears in the reconstruction.

## Heading Mirror Check

RECONSTRUCTION.md headings are:

- System-level intent
- Per-feature whys
- Scope
- Architecture
- Data model
- API surface
- Simulation engine design
- Sync model
- Frontend rendering pipeline
- Audio pipeline
- Accessibility surfaces
- Performance budgets and observability
- Rollout
- Risks

These headings mirror the PLAN structure, not GOLD_WHYS.md. They do not echo gold-list section titles such as "System-level whys," "Feature-level whys," "Complete features list," or the F/S identifiers. The first two headings are expected by the reconstruction format and are not evidence of contamination.

## 1:1 Mapping Suspect Check

Verdict: no 1:1 gold mapping pattern.

The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. It follows the candidate PLAN sections and includes many plan features outside the 40 gold feature whys. Several gold targets have no neat corresponding reconstruction item, and several reconstruction bullets combine multiple plan mechanisms. This is plan-mirroring, not gold-list mirroring.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "The client never writes personality state directly, the server authors all deltas, and canonical state is the absolute source of truth so there is no last-write-wins." | PLAN Architecture says the client never writes personality state directly; Sync model says server is absolute source of truth; Conflict Prevention says clients do not push absolute personality state. | Supported. |
| "The risk section says counting minimized tabs makes drift happen too fast and breaks the alive over weeks illusion." | PLAN Risks: "If presence is calculated poorly (e.g., counting minimized tabs), drift happens too fast, breaking the alive over weeks illusion." | Supported. |
| "The plan rejects a generic ARIA state-change log and says cheap ARIA labels destroy charm." | PLAN Accessibility says narration is not a generic ARIA state-change log; Risks says cheap ARIA labels destroy product charm. | Supported. |

## Verdict

PASS.

The frozen reconstruction reads as plan-derived. It contains no gold IDs, no rubric vocabulary, no gold-order enumeration, and its strongest sentences can be traced directly to PLAN passages. Minor synthesized labels such as "bounded, calm social presence" are ordinary summaries of plan content rather than contamination signatures.
