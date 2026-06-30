# VALIDITY_AUDIT - CARE run 001

## ID Leakage Check

Verdict: no leakage found. A targeted search of the frozen reconstruction found no gold IDs or rubric IDs such as `S1`, `F1`, `F40`, or `R-F01`. The same targeted search of the plan also returned no such IDs, so there were no suspicious ID matches to cross-reference.

## Vocabulary Check

Sampled load-bearing phrases from `RECONSTRUCTION.md` and compared them to `PLAN.md`:

| Reconstruction phrase | Present in or directly derived from PLAN? | Note |
|---|---|---|
| `client is a pure renderer of snapshots` | yes | Exact plan language in scope/architecture. |
| `read-many/write-one architecture` | yes | Exact plan phrase in Sync model. |
| `drift never moves down on neglect` | yes | Exact plan rationale around `max(0, delta)`. |
| `no recorded audio, ever` | yes | Exact scope/audio rule in plan. |
| `naturalist prose, not state-list` | yes | Exact accessibility/narration framing in plan. |
| `hard failure, not a warning` | yes | Exact CI budget language in plan. |
| `aggregate-only observability` | yes | Exact privacy/observability framing in plan. |
| `synthetic aviary` | yes | Exact calibration harness phrase in plan. |
| `different product decision` | yes | Exact social scope-creep language in plan. |
| `felt-aliveness pacing only` | yes | Exact drift-risk mitigation language in plan. |

No rubric-side phrases such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `gold`, or `rubric` appeared in the reconstruction.

## Heading Mirror Check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope`
- `### 2. Architecture`
- `### 3. Data model`
- `### 4. API surface`
- `### 5. Simulation engine design`
- `### 6. Sync model`
- `### 7. Frontend rendering pipeline`
- `### 8. Audio pipeline`
- `### 9. Accessibility surfaces`
- `### 10. Performance budgets and observability`
- `### 11. Rollout`
- `### 12. Risks`

The `###` headings mirror the plan's section structure, not `GOLD_WHYS.md` headings. The two top-level headings are expected reconstruction output structure and do not mirror the gold list's S/F taxonomy.

## 1:1 Mapping Suspect Check

No near-1:1 mapping to the gold list was found. The reconstruction has nine system bullets, but they are not S1-S9 in gold order: it starts with server-canonical state (gold S7-like), additive/non-punitive interaction, gamification/Tamagotchi restraint, and procedural aliveness. The per-feature section follows the plan's twelve implementation sections and includes many plan-specific implementation choices, plus several `NOT RECOVERABLE FROM PLAN` rows. It does not enumerate F1-F40 or use gold ordering.

## Plan-Derivation Spot Check

1. Reconstruction: `The plan treats this as the concrete implementation of the no last-write-wins rule.`
   Plan support: Section 2 states the simulation tick is the only writer of canonical state and calls this the implementation of the no-last-write-wins rule; section 6 expands the same read-many/write-one architecture.

2. Reconstruction: `The rationale is that calls should vary every time, avoid stacked recorded loops, avoid audio files in the bundle, and honor no recorded-audio fallback, ever.`
   Plan support: Sections 1, 5, and 8 specify procedural client-side WebAudio, no recorded audio, a shared graph for chorus mixing, lazy motif libraries, and silent WebAudio fallback with captions.

3. Reconstruction: `The rationale for DB permission enforcement is that code-review discipline can erode under deadline pressure.`
   Plan support: Section 12 explicitly names sync correctness risk from rushed changes and mitigates it by enforcing DB permissions rather than relying on code review.

All three articulate sentences are directly derivable from plan passages.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it mirrors the plan's headings, reuses plan vocabulary, contains no gold/rubric identifiers, and does not align in a neat S1-S9/F1-F40 sequence. The main scoring issue is omission and rule-without-why compression, not contamination.
