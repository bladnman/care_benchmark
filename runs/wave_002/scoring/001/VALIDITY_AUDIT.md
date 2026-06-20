# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction shows no significant contamination signatures: it uses no gold IDs, no rubric taxonomy, and its structure mirrors the PLAN sections rather than the gold list. A few articulate synthesis phrases go beyond exact PLAN wording, but they are reasonable plan-derived summaries rather than gold-side leakage.

## ID Leakage

No gold IDs or runner-only identifiers were found in `RECONSTRUCTION.md`. Search hits for `S1`-`S9`, `F1`-`F40`, `R-F`, `gold`, `rubric`, `multi-layer`, `feature-level fidelity`, `intent fidelity`, `weight-3`, and `load-bearing` produced no leakage hits. The only repeated special phrase was `NOT RECOVERABLE FROM PLAN`, which is part of the reconstruction workflow rather than a gold identifier.

PLAN cross-check: there was no need to cross-reference leaked IDs because no leaked IDs appeared in RECONSTRUCTION.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `Feels alive, not robotic` | Exact phrase appears in PLAN Scope. | clean |
| `birds appear mid-action` | Exact phrase appears in PLAN Frontend Rendering Pipeline. | clean |
| `never identical recordings` | Exact phrase appears in PLAN Simulation/Audio sections. | clean |
| `canonical, slow, and server-authored` | PLAN says server owns all simulation state, slow tick, canonical state. | plan-derived synthesis |
| `quiet, opt-in, and read-only` | PLAN says quiet opt-in read-only viewing. | clean |
| `same aviary, different rendering intensity` | Exact phrase appears in PLAN Accessibility/Frontend. | clean |
| `part of the aviary, not an alternate product` | Not exact; supported by naturalist narration, reduced motion as same aviary, captions, and `not checklist` phase gate. | plan-derived synthesis |
| `privacy and telemetry are deliberately separated` | PLAN says database separated from telemetry and telemetry pipelines never access simulation database. | plan-derived synthesis |
| `Performance is a product constraint, not a late optimization` | Not exact; supported by explicit budgets and phase gates. | mild synthesis, not suspicious |

No rubric-only vocabulary such as `multi-layer`, `weight-3`, `feature-level fidelity`, or `load-bearing` appears in the reconstruction.

## Heading Mirror

RECONSTRUCTION headings:

| Reconstruction heading | Nearest PLAN heading | Gold-list mirror concern |
|---|---|---|
| `## System-level intent` | Phase 2A required section shape, not a PLAN heading. | expected, not a gold mirror |
| `## Per-feature whys` | Phase 2A required section shape. | expected, not a gold mirror |
| `### Scope` | `## Scope` | mirrors PLAN |
| `### Architecture` | `## Architecture` | mirrors PLAN |
| `### Data model and API surface` | `## Data Model` + `## API Surface` | mirrors PLAN grouping |
| `### Simulation engine design` | `## Simulation Engine Design` | mirrors PLAN |
| `### Sync model` | `## Sync Model` | mirrors PLAN |
| `### Frontend rendering pipeline` | `## Frontend Rendering Pipeline` | mirrors PLAN |
| `### Audio pipeline` | `## Audio Pipeline` | mirrors PLAN |
| `### Performance budgets and observability` | `## Performance Budgets and Observability` | mirrors PLAN |
| `### Rollout` | `## Rollout` | mirrors PLAN |

The headings do not echo `GOLD_WHYS.md` sections such as system IDs, feature IDs, or file-grouped F1-F40 headings. They follow the PLAN layout.

## 1:1 Mapping Suspect

No. RECONSTRUCTION does not enumerate S1-S9 or F1-F40, and it does not follow the gold feature order. It walks the PLAN sections and emits many rows that are not gold-scored features, such as CDN snapshots, WebSocket streaming, magic-link endpoints, WebGL/2D canvas, Redux/Redux-Saga, and rollout instrumentation. This is consistent with plan derivation rather than a neat gold-target mapping.

## Plan-Derivation Spot Check

1. Reconstruction sentence: `The aviary is canonical, slow, and server-authored.`
   PLAN support: Architecture says `Server owns all simulation state`; Simulation Engine says the server-side tick `Runs ~once/minute`; Sync Model says `Single source of truth: server-side canonical state`. Verdict: supported.

2. Reconstruction sentence: `Accessibility is part of the aviary, not an alternate product.`
   PLAN support: Accessibility includes naturalist screen-reader prose, cross-fade reduced motion as `Same aviary, different visual intensity`, captions, keyboard navigation, and a phase gate requiring accessibility surfaces `fully functional (not checklist)`. Verdict: supported synthesis.

3. Reconstruction sentence: `Privacy and telemetry are deliberately separated from simulation state.`
   PLAN support: Architecture says the database has account/per-bird state `separated from telemetry`; Privacy Boundary says telemetry pipelines `never access simulation database`; Observability says `Aggregate-only telemetry`. Verdict: supported.

## Verdict Rationale

PASS. The reconstruction contains normal synthesis, but no gold identifiers, no rubric vocabulary, no gold-heading mirror, and no neat one-to-one gold mapping. The strongest evidence is structural: the reconstruction follows PLAN sections and includes many non-gold implementation rows, including honest `NOT RECOVERABLE FROM PLAN` entries.
