# VALIDITY_AUDIT — CARE run 001

## ID Leakage

PASS. Mechanical search found no gold IDs or rebuild IDs in the frozen reconstruction: no `F1`-style, `S1`-style, `R-F01`, or `R-S01` tokens appeared. The same search in the assigned PLAN also returned no hits, so there are no suspicious ID matches to cross-reference.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | PLAN support | Verdict |
|---|---|---|
| `one web aviary per magic-link account` | PLAN §1 uses the same phrase. | plan-derived |
| `two stable-identity starter birds` | PLAN §1 uses the same phrase. | plan-derived |
| `lowercase, present-tense naturalist prose` | PLAN §1 uses the same phrase. | plan-derived |
| `database is the sole canonical store` | PLAN §2 uses the same phrase. | plan-derived |
| `never infer presence from an open tab` | PLAN §5 uses the same phrase. | plan-derived |
| `never subtract from persisted traits` | PLAN §5 uses the same phrase. | plan-derived |
| `first bird visible within 500 ms` | PLAN §8 uses the same target. | plan-derived |
| `complete accessible render/audio alternatives` | PLAN §1 uses the same phrase. | plan-derived |
| `same canonical bird/day/weather projection` | PLAN §4 uses the same phrase. | plan-derived |
| `scores, notifications, visible trait values or punitive absence behavior` | PLAN §9 uses the same constraint. | plan-derived |

Mechanical scorer-side vocabulary search found no hits for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

## Heading Mirror

The reconstruction headings are exactly:

- `## System-level intent`
- `## Per-feature whys`

These headings are required by the phase-2A prompt and do not mirror the gold-list section titles beyond the intended output contract. There are no S1-S9 or F1-F40-style headings and no section ordering that mirrors GOLD_WHYS.md.

## 1:1 Mapping Suspect

PASS. The reconstruction does not create 9 system entries followed by 40 feature entries in gold order. Instead, it follows the PLAN's own structure: product contract, system boundaries, persistence, API contracts, presence/simulation, rendering, audio/accessibility, delivery, and risks. Many per-feature bullets are plan-order implementation bullets, including many features that are not gold-why anchors. This is consistent with plan derivation rather than held-out target mapping.

## Plan-Derivation Spot Check

1. Reconstruction: "Presence is a validated input for slow character drift, not a public engagement counter." PLAN support: §5 says presence qualifies via visible/focus/recent input, is "validation for drift integrity, not a public engagement counter," and drives expressive traits. Verdict: grounded.

2. Reconstruction: "Privacy boundaries are product design, not only storage design." PLAN support: §§2-3 separate private snapshots, encrypted email, UUID-only references and aggregate-only metrics; §§8-9 add payload audits and no per-account engagement dashboards. Verdict: grounded synthesis.

3. Reconstruction: "Accessibility must be an equivalent aviary experience, not a reduced status page." PLAN support: §§1,6-8 require complete accessible alternatives, reduced-motion preserving mood/calls/notebook/drift, naturalist narration, keyboard/focus controls and early accessibility review. Verdict: grounded synthesis.

## Verdict

PASS. The frozen reconstruction reads as plan-derived: no ID leakage, no rubric/gold vocabulary, no gold-order mapping, and the most articulate synthesis sentences are grounded in multiple passages of the assigned PLAN.
