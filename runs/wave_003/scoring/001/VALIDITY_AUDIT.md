# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A search for gold IDs and rubric markers (`S1`-`S9`, `F1`-`F40`, `R-FNN`, `weight-3`, `feature-level fidelity`, `multi-layer recovery`, `load-bearing`, `intent fidelity`) found no hits in the frozen reconstruction. The reconstruction does not use the gold IDs, PRD filenames as a taxonomy, or the scorer-side why labels.

## Vocabulary Check

Sampled reconstruction phrases and plan grounding:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `slow, observational relationship` | PLAN §1 uses the same phrase. | plan-derived |
| `idle attention is real interaction` | PLAN §1 and presence accounting use the same principle. | plan-derived |
| `server-side canonical simulation tick` | PLAN §§1, 6, and 7 repeat the phrase. | plan-derived |
| `sole writer` | PLAN §7.1 states the server tick is the sole writer. | plan-derived |
| `Strict Privacy Firewall` | PLAN §11.2 uses this heading. | plan-derived |
| `Naturalist Voice` / `Matter-of-Fact Voice` | PLAN §2.3 names both registers. | plan-derived |
| `raw state dumps` and `debugging tool` | PLAN risk matrix uses both terms. | plan-derived |
| `canned, repetitive audio loops` | PLAN §9.3 uses this phrase. | plan-derived |
| `small flock` | PLAN §1 and lifecycle framing support this wording. | plan-derived |
| `quiet and non-invasive` | PLAN uses optional/quiet social visits and no-notification language. | plan-derived semantic compression |

No sampled phrase reads like scorer-side rubric vocabulary. I did not find `multi-layer`, `weight-3`, `feature-level fidelity`, or `intent fidelity` in the reconstruction.

## Heading Mirror Check

Reconstruction headings:

| Reconstruction heading | Gold-list / rubric comparison | Assessment |
|---|---|---|
| `## System-level intent` | Generic phase-2A output section, not a gold heading. | not suspicious |
| `## Per-feature whys` | Generic reconstruction section, not ordered as F1-F40. | not suspicious |
| `### Scope & Boundary Definitions` | Mirrors PLAN §2, not GOLD_WHYS.md. | plan-derived |
| `### System Architecture & Component Topography` | Mirrors PLAN §3. | plan-derived |
| `### Canonical Data Model & Schema Specifications` | Mirrors PLAN §4. | plan-derived |
| `### API Surface & Inter-Service Protocol Contracts` | Mirrors PLAN §5. | plan-derived |
| `### Simulation Engine & Mathematical Drift Architecture` | Mirrors PLAN §6. | plan-derived |
| `### Multi-Device Synchronization & Conflict Prevention Model` | Mirrors PLAN §7. | plan-derived |
| `### Frontend Rendering Pipeline & Visual Scene Architecture` | Mirrors PLAN §8. | plan-derived |
| `### WebAudio Procedural Synthesis & Soundscape Engine` | Mirrors PLAN §9. | plan-derived |
| `### Accessibility Architecture & Inclusive Design Surfaces` | Mirrors PLAN §10. | plan-derived |
| `### Performance Budgets, Resource Constraints & Observability` | Mirrors PLAN §11. | plan-derived |
| `### Rollout Strategy, Calibration Framework & Progression Pacing` | Mirrors PLAN §12. | plan-derived |

The heading structure follows the candidate PLAN, not the gold files or S/F ordering.

## 1:1 Mapping Suspect Check

Verdict: not suspect. The system section has 10 free-form bullets rather than 9 S1-S9 items, and they are not in gold order. The per-feature section follows the PLAN's architecture sections and implementation bullets; it does not enumerate F1-F40, does not use feature IDs, and leaves some plan bullets as `NOT RECOVERABLE FROM PLAN`. This is consistent with a plan-derived reconstruction rather than a gold-list mirror.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `The server split ensures the aviary continues living even when the user is away...` | PLAN §1 says the server/client split ensures the aviary continues living even when the user is away; PLAN §8.2 says the first frame should be already in motion. | supported |
| `Privacy is structural, not only policy text.` | PLAN §3.1 names telemetry isolation; PLAN §11.2 states a strict privacy firewall and excludes per-bird data from analytics/ML. | supported |
| `Accessibility is part of the aviary voice, not a diagnostic layer.` | PLAN §2.1 calls accessibility first-class; PLAN §10 uses naturalist narration/captions/reduced motion; PLAN risk matrix rejects raw state dumps as a debugging tool. | supported |

## Verdict: PASS

The reconstruction reads as derived from the assigned PLAN. I found no gold ID leakage, no rubric vocabulary leakage, no 1:1 S/F mapping, and no gold-heading mirror. The reconstruction's strongest phrases are traceable to PLAN wording or close semantic compression of PLAN sections.
