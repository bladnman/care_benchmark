# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction reads as derived from the PLAN, not from the gold list or rubric. I found no gold ID leakage, no rubric vocabulary hits, no gold-section heading mirror, and no neat S1-S9/F1-F40 mapping.

## ID Leakage Check

Search result: no hits for gold-style IDs or external taxonomy patterns such as `S1`, `F1`, `F40`, `R-F01`, `multi-layer`, `weight-3`, `intent fidelity`, or `gold why` in the frozen reconstruction or PLAN.

No offending IDs found. Because no gold IDs appear in RECONSTRUCTION, there is no absent-from-PLAN ID leakage to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "felt continuity" | PLAN sec 1 says optimize for felt continuity. | Plan-derived. |
| "canonical server continuity" | PLAN core invariant: clients never mutate canonical state; simulation service only writer. | Plan-derived phrasing. |
| "individual bird recognizability" | PLAN sec 1 explicitly optimizes for individual bird recognizability. | Exact/near-exact plan vocabulary. |
| "quiet, non-gamified product behavior" | PLAN excludes gamification and says APIs must maintain product quietness. | Plan-derived synthesis. |
| "matter-of-fact system prose" | PLAN product voice boundary uses this phrase. | Exact plan vocabulary. |
| "Security and privacy are architectural requirements" | PLAN sec 9 uses the sentence directly. | Exact plan vocabulary. |
| "Accessibility Regression Into Fallback Product" | PLAN risk heading uses this phrase. | Exact plan vocabulary. |
| "Tab open alone never counts" | PLAN presence section uses this sentence. | Exact plan vocabulary. |
| "first visible frame feels like the aviary was already running" | PLAN definition of done uses this phrase. | Exact plan vocabulary. |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN, but this is phase-2A reconstruction format, not gold/rubric content. | Not a contamination signature by itself. |

No suspicious rubric-side terms such as "feature-level fidelity", "intent fidelity", "weight-3", "gold why", or "load-bearing" appear in RECONSTRUCTION.

## Heading Mirror Check

RECONSTRUCTION headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Product Boundary and Scope`
- `### Architecture Overview and Data Model`
- `### API Surface`
- `### Simulation Engine`
- `### Frontend Rendering Pipeline and Interaction UI`
- `### Accessibility Plan`
- `### Performance, Observability, Privacy, Rollout, and Risk Controls`

These headings mirror the PLAN's implementation sections, often by merging adjacent plan sections. They do not mirror GOLD_WHYS section headings or the S/F taxonomy. The first two headings are expected reconstruction output structure, not gold leakage.

## 1:1 Mapping Suspect Check

No 1:1 gold mapping found. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve the gold order, and contains many plan-derived items outside the 49 gold whys, such as API endpoints, database records, route choices, and rollout/testing controls. It is organized around PLAN sections rather than the gold list.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan's core invariant is that 'Clients never mutate personality, mood, bird positions, or notebook entries directly.'"
   - PLAN support: sec 2 core invariant says exactly that clients never mutate those state fields directly.
   - Assessment: directly plan-derived.

2. Reconstruction: "Presence means actual host attention, not an open tab."
   - PLAN support: sec 5 says the detector requires visible, focus, and recent activity, and says "Tab open alone never counts."
   - Assessment: plan-derived synthesis.

3. Reconstruction: "Accessibility is part of the same product, not a fallback."
   - PLAN support: sec 7 says accessibility must ship in v1 and be built from the same canonical state; risk section names "Accessibility Regression Into Fallback Product."
   - Assessment: plan-derived synthesis.

## Verdict Rationale

PASS. The reconstruction's vocabulary, headings, and item order track the PLAN closely. It contains no gold IDs, no rubric terms, and no suspicious one-to-one gold-target mapping. The strongest non-PLAN phrase, `NOT RECOVERABLE FROM PLAN`, is a reconstruction-format marker rather than a gold-list leak.
