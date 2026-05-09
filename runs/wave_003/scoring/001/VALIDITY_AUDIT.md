# VALIDITY_AUDIT — CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A targeted regex search of the frozen reconstruction found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, canonical-number, rubric, gold, multi-layer, weight-tier, feature-level-fidelity, or intent-fidelity markers. The only notable vocabulary hit was `load-bearing`, and the same phrase appears in PLAN.md, so it is plan-derived rather than gold-side leakage.

Hits reviewed:

| Candidate hit | Reconstruction sentence | PLAN cross-check | Finding |
|---|---|---|---|
| `load-bearing` | "The product's affective core is load-bearing." | PLAN.md opening uses the same phrase. | Not leakage. |
| `load-bearing` | "The plan calls the asymmetry rule 'load-bearing'." | PLAN.md §5.2 labels the asymmetry rule load-bearing. | Not leakage. |

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "affective core is load-bearing" | yes, opening paragraph | Plan-derived. |
| "server-authoritative" | yes, summary/architecture | Plan-derived. |
| "does not run the drift function as a preview" | yes, §2.2 | Plan-derived. |
| "voice cannot fork" | yes, §9.1 | Plan-derived. |
| "not parity-by-checklist" | yes, §9 | Plan-derived. |
| "central conceit cracks" | yes, risk §12.8 | Plan-derived. |
| "network and credentials boundary" | yes, §2.4/§3.9 | Plan-derived. |
| "feature-level fidelity" / "intent fidelity" / "multi-layer" / "weight-3" | no hits in reconstruction | No rubric vocabulary leakage. |

## Heading Mirror Check

Reconstruction headings are plan-section-like, not gold-list-like:

| Reconstruction heading | Closest gold/rubric section | Finding |
|---|---|---|
| `## System-level intent` | Required reconstruction structure, not a gold title. | OK. |
| `## Per-feature whys` | Required reconstruction structure, not a gold title. | OK. |
| `### Scope and non-goals` | PLAN §1 / non-goals content. | Plan-derived. |
| `### Architecture and storage` | PLAN §§2-3. | Plan-derived. |
| `### API and user-visible surfaces` | PLAN §4. | Plan-derived. |
| `### Simulation engine` | PLAN §5. | Plan-derived. |
| `### Sync model` | PLAN §6. | Plan-derived. |
| `### Frontend rendering pipeline` | PLAN §7. | Plan-derived. |
| `### Audio pipeline` | PLAN §8. | Plan-derived. |
| `### Accessibility surfaces` | PLAN §9. | Plan-derived. |
| `### Performance budgets and observability` | PLAN §10. | Plan-derived. |
| `### Rollout and operability` | PLAN §11. | Plan-derived. |
| `### Risks and intentionally open seams` | PLAN §§12-13. | Plan-derived. |

No headings mirror the gold S1-S9/F1-F40 titles or their order.

## 1:1 Mapping Suspect Check

No 1:1 gold-list mapping found. The reconstruction has 12 system-level bullets and many plan-section bullets, not 9 system entries plus 40 feature entries in gold order. Several gold targets are absent or explicitly `NOT RECOVERABLE FROM PLAN` (for example account export and account deletion), which also argues against leakage from the gold list. The ordering follows the PLAN structure: scope, architecture/storage, API, simulation, sync, frontend, audio, accessibility, performance, rollout, risks.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The client is a fast, small renderer, not a simulator." | PLAN §2.2 says the client owns rendering/audio/interpolation/input and does not run drift, project mood, or predict call grammar. | Supported. |
| "The product refuses engagement pressure." | PLAN §1 forbids streaks/XP/badges/notifications; §11.3 rejects DAU/MAU; §12.6 discusses engagement-feature creep. | Supported. |
| "Accessibility is a first-class designed surface." | PLAN §9 says accessibility is a designed surface, not parity-by-checklist; §7.5 and §12.4 define reduced-motion and regression safeguards. | Supported. |

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no gold IDs, no rubric-only vocabulary, no gold-heading mirror, no suspicious 1:1 S/F mapping, and sampled articulate sentences have direct PLAN support. The few high-salience terms that could look benchmarky are present in the PLAN itself.
