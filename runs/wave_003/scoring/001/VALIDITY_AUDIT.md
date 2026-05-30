# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as derived from the PLAN, not from the gold list or rubric. It mirrors the PLAN section order and vocabulary, contains no gold IDs, and its omissions are consistent with a reconstructor limited to the compact plan.

## ID leakage

No gold ID leakage found. A search of `RECONSTRUCTION.md` for `S1`-`S9`, `F1`-`F40`, `R-F`, `weight-3`, `multi-layer`, `feature-level`, `intent fidelity`, and `planning quality` returned no hits. The reconstruction uses numbered bullets and plan-section headings, but not benchmark IDs or rubric labels.

## Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "observational relationship-building" | PLAN.md:4 exact phrase | Plan-derived. |
| "single-user virtual aviary" | PLAN.md:4 exact phrase | Plan-derived. |
| "No streaks, levels, achievements, or scores" | PLAN.md:16 exact phrase | Plan-derived. |
| "Procedural aliveness" | PLAN.md:7-8 procedural behavior/calls and PLAN.md:162/168 aliveness language | Synthesized from plan, not gold/rubric vocabulary. |
| "server is the sole authority" | PLAN.md:25 exact phrase | Plan-derived. |
| "The tick is the only process allowed" | PLAN.md:89 exact phrase | Plan-derived. |
| "clients only send signals" | PLAN.md:109 exact phrase | Plan-derived. |
| "Naturalist voice" / "naturalist observations" | PLAN.md:9, 98, 135-141 | Plan-derived. |
| "Accessibility is not a fallback" | PLAN.md:169 exact risk phrasing | Plan-derived. |
| "Privacy Boundary" / "No per-user or per-bird data" | PLAN.md:157-158 | Plan-derived. |

No rubric-side terms such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `load-bearing`, or `intent fidelity` appear in the reconstruction.

## Heading mirror

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data Model`
- `### 4. API Surface`
- `### 5. Simulation Engine Design`
- `### 6. Sync Model`
- `### 7. Frontend Rendering Pipeline`
- `### 8. Accessibility Surfaces`
- `### 9. Performance Budgets and Observability`
- `### 10. Rollout`

These mirror the PLAN's own section headings, not the gold-list groupings. The only generic headings, `System-level intent` and `Per-feature whys`, are expected reconstruction-output structure and do not echo gold section titles closely enough to flag.

## 1:1 Mapping Suspect

No near-1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction has seven system-level bullets, not nine, and the per-feature section follows PLAN sections and plan bullets rather than gold IDs or gold order. It includes many plan-only items such as `Auth Service`, `Snapshot Service`, `HTML5 Canvas or WebGL visual layer`, and rollout phases, which would be unlikely if it were mapping directly to the gold whys.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Result |
|---|---|---|
| "Server-side canonical state and simulation authority..." | PLAN.md:25 says the server is sole authority; PLAN.md:89 says the tick is the only mutator; PLAN.md:107-110 defines the source of truth and additive deltas. | Supported. |
| "Naturalist voice as both experience and access surface..." | PLAN.md:9 names naturalist field notebook; PLAN.md:98 naturalist observation entry; PLAN.md:135-141 naturalist screen-reader prose and call captions. | Supported. |
| "Accessibility, performance, and privacy are designed boundaries..." | PLAN.md:12 accessibility in scope; PLAN.md:149-158 performance/observability budgets; PLAN.md:169 warns against accessibility as fallback. | Supported. |

## Verdict Rationale

PASS: the reconstruction is compact and sometimes interpretive, but the interpretive language is traceable to the PLAN. There are no benchmark IDs, no rubric vocabulary, no gold-heading mirror, and no suspicious gold-order enumeration.
