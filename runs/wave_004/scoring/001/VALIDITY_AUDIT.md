# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found.

The frozen reconstruction does not use gold IDs such as `S1`-`S9`, `F1`-`F40`, `R-F01`, or a gold-list taxonomy. It uses the required broad headings `System-level intent` and `Per-feature whys`, then follows the PLAN's own section order (`Scope`, `Architecture`, `Data model`, etc.). I found no gold ID that appears in RECONSTRUCTION while absent from PLAN.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Suspect? |
|---|---|---|
| `server-canonical honesty` | PLAN repeatedly says server-owned/canonical state and server-only writer. | no |
| `architectural absence` | PLAN section 9.5 uses this exact phrase for privacy defense. | no |
| `affective-perf bridge` | PLAN section 9.1 uses this exact phrase for the 500ms target. | no |
| `actual product, not a stripped-down fallback` | PLAN section 8 and section 15 use this accessibility framing. | no |
| `soloable tracks` | PLAN section 7.4 uses this for listen-in failure. | no |
| `the bird becomes a number and the relationship collapses` | PLAN section 15 uses this rationale for hidden vectors. | no |
| `designed surface, not a checklist` | PLAN section 8 uses this accessibility phrase. | no |
| `load-bearing` | PLAN uses this term throughout; it is not gold-only vocabulary here. | no |
| `rule-without-why`, `multi-layer recovery`, `feature-level fidelity` | Not present in RECONSTRUCTION. | no |

The vocabulary reads plan-derived. Several phrases are highly diagnostic, but they are diagnostic of the PLAN, not the rubric/gold side.

## Heading Mirror Check

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine design`
- `### 6. Frontend rendering pipeline`
- `### 7. Audio pipeline`
- `### 8. Accessibility surfaces`
- `### 9. Performance budgets and observability`
- `### 10. Sync model`
- `### 11. Rollout`
- `### 12. Risks`
- `### 13. Open questions resolved`
- `### 14. What the team will build first`
- `### 15. Summary of load-bearing decisions`

The `###` headings mirror PLAN section headings, not GOLD_WHYS section titles. The two `##` headings are the expected reconstruction format described to phase 2A. No near-exact gold heading mirror is present.

## 1:1 Mapping Suspect Check

Verdict: not suspect.

The reconstruction does not enumerate S1-S9 or F1-F40 in order. It lists 11 system-level principles rather than 9, and the per-feature section follows PLAN structure with many more than 40 bullets, including multiple `NOT RECOVERABLE FROM PLAN` entries for ordinary plan items. This is not a neat mapping to the gold list.

## Plan-Derivation Spot Check

1. Reconstruction: `Privacy is not just policy in the plan; it is designed as missing fields, missing tables, missing roles, and missing access.`
   PLAN support: section 3.10 narrowly defines aggregate telemetry; section 3.11 says no global tables; section 9.5 says metrics are not defined and the data pipeline lacks simulation DB access.

2. Reconstruction: `The plan ties bundle size, time to first bird visible <500ms, 60fps, no memory growth, and CDN/static delivery to felt aliveness.`
   PLAN support: section 9.1 lists those budgets and calls the 500ms target the affective-perf bridge; section 2.4 includes CDN edge delivery.

3. Reconstruction: `Reduced-motion mode changes the visual register while keeping simulation, calls, mood, drift, and notebook behavior intact.`
   PLAN support: section 6.7 says reduced motion changes pose/flight/ambient motion while calls still play, birds still drift, mood still changes, and notebook still notices things.

All three articulate sentences are grounded in the PLAN.

## Verdict: PASS

No significant contamination signatures found. The reconstruction reads as plan-derived: it mirrors PLAN section order, uses PLAN vocabulary, lacks gold IDs/rubric vocabulary, and does not form a 1:1 map to S1-S9 or F1-F40.
