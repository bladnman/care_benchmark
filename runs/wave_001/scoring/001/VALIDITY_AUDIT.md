# VALIDITY_AUDIT — CARE run 001

## 1. Gold ID Leakage Check

Verdict: PASS. The frozen reconstruction does not use gold IDs such as `S1`-`S9`, `F1`-`F40`, `R-F01`, or rubric-like per-why identifiers. Its headings and bullets are organized around the candidate PLAN sections: `Executive Summary & Product Scope`, `System Architecture & Component Boundaries`, `Data Model & Storage Schema`, and so on. Because the PLAN also uses those headings, this is plan-derived structure rather than gold-list leakage.

Observed ID-like tokens in the reconstruction are ordinary implementation labels from the PLAN, such as `60-second`, `2D`, `30-60s`, and `<500ms`, not gold identifiers.

## 2. Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| `ambient, observational relationships` | yes, PLAN §1.1 | Plan-derived. |
| `idle attention is real interaction` | yes, PLAN §1.1 | Plan-derived. |
| `monotonic asymmetry` | yes, PLAN §5.3 | Plan-derived. |
| `Server is the Sole Canonical Writer` / `Server-authored canonical reality` | yes, PLAN §6.1 supports it | Plan-derived paraphrase. |
| `Naturalist Register` / `Matter-of-Fact Register` | yes, PLAN §1.4 | Plan-derived. |
| `STRICT DATA ISOLATION BOUNDARY` | yes, PLAN §10.2 | Plan-derived. |
| `reduced-motion cross-fade mode (not just disabled animations)` | yes, PLAN §1.2 and §7.4 | Plan-derived. |
| `rule_without_why`, `feature-level fidelity`, `weight-3`, `load-bearing`, `intent fidelity` | no occurrences in reconstruction | No rubric-side vocabulary leakage. |

The vocabulary reads like a reconstruction from the implementation plan. One phrase, `avoiding a user-authored journal or achievement feed`, is more interpretive than the PLAN, and I treated it as an ungrounded reconstruction for F33 scoring, but it is not gold-ID or rubric leakage.

## 3. Heading Mirror Check

| Reconstruction heading | Source match | Assessment |
|---|---|---|
| `## System-level intent` | Required reconstruction format, not a gold heading | Acceptable. |
| `## Per-feature whys` | Required reconstruction format, not a gold heading | Acceptable. |
| `### Executive Summary & Product Scope` | Mirrors PLAN §1 | Plan-derived. |
| `### System Architecture & Component Boundaries` | Mirrors PLAN §2 | Plan-derived. |
| `### Data Model & Storage Schema` | Mirrors PLAN §3 | Plan-derived. |
| `### API Surface & Wire Protocols` | Mirrors PLAN §4 | Plan-derived. |
| `### Simulation Engine & Mathematical Models` | Mirrors PLAN §5 | Plan-derived. |
| `### Sync Model & Conflict Avoidance` | Mirrors PLAN §6 | Plan-derived. |
| `### Frontend Rendering Pipeline` | Mirrors PLAN §7 | Plan-derived. |
| `### Audio Pipeline & Procedural Soundscape` | Mirrors PLAN §8 | Plan-derived. |
| `### Accessibility Architecture` | Mirrors PLAN §9 | Plan-derived. |
| `### Performance Budgets, Optimization & Observability` | Mirrors PLAN §10 | Plan-derived. |
| `### Rollout Strategy, Verification & Growth Ramp` | Mirrors PLAN §11 | Plan-derived. |
| `### Risk Analysis & Mitigation Matrix` | Mirrors PLAN §12 | Plan-derived. |

No headings mirror `GOLD_WHYS.md` section names or the S/F taxonomy. The heading mirror is to the PLAN, not the gold list.

## 4. 1:1 Mapping Suspect Check

Verdict: PASS. The reconstruction does not provide neat S1-S9 or F1-F40 rows, does not preserve gold-list order, and includes many plan-specific implementation rows that are not gold why anchors: Edge API Gateway, magic links table, active sessions table, reconnection state machine, WebAudio graph, test harnesses, and risk mitigations. This argues against a 1:1 gold mapping.

There is a broad system-intent bullet list of eight items, but it is not the nine gold system whys and omits/restates several gold categories differently. The per-feature section follows PLAN section order, not GOLD_WHYS order.

## 5. Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| `The plan's mission says Pocket Aviary is "ambient" and built around "low-key, observational relationships" with procedural birds.` | PLAN §1.1 says the product is an ambient browser aviary built around low-key observational relationships. | Supported. |
| `The simulation tick runs on an independent worker cluster every 60 seconds per aviary, irrespective of client connection status.` | PLAN §5.1 states a 60-second tick per aviary irrespective of client connection status. | Supported. |
| `The rationale is preventing annoyance by throttling idle narration and prioritizing user actions with polite queuing and concise naturalist prose.` | PLAN §9.1 gives 30-60s cadence and user-action priority; PLAN §12 names screen-reader flooding/annoyance mitigation. | Supported. |
| `The notebook is ... avoiding a user-authored journal or achievement feed.` | PLAN states sparse, read-only, auto-generated notebook entries, but does not explicitly give the journal/achievement-feed rationale. | Ungrounded inference for scoring, but isolated and not a contamination signature. |

## 6. Verdict

PASS. The reconstruction shows no significant contamination signatures: no gold IDs, no rubric vocabulary, no gold-heading mirror, and no neat S/F mapping. It mostly tracks PLAN headings and wording. One interpretive sentence about the notebook rationale exceeded the PLAN evidence, so I treated it as ungrounded in scoring, but the pattern is ordinary reconstruction inference rather than proof of gold-list contamination.
