# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

**Result: PASS.** I found no gold IDs such as F1-F40, S1-S9, R-F IDs, or rubric section identifiers in the frozen reconstruction. The reconstruction uses plan-section language such as "Scope," "Architecture," "Data Model," and "Risks," not the gold taxonomy.

No leakage hits.

## Vocabulary Check

Sampled phrases from RECONSTRUCTION and checked them against PLAN grounding:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Anti-gamification, age-over-attention care" | PLAN scope/non-goals and ambiguity table: no gamification; age-not-attention unlocks birds | Plan-derived |
| "Watching without moving is the product" | PLAN ambiguity table uses the same phrase | Exact plan-derived phrase |
| "Aviary as place, not app" | PLAN risk says the product can collapse from place to app | Plan-derived |
| "Server-owned inner life, thin renderers" | PLAN sync model says server holds the only canonical copy and clients are thin renderers | Plan-derived |
| "Naturalist voice over app voice" | PLAN naturalist prose surfaces and matter-of-fact timeout error | Plan-derived paraphrase |
| "Accessibility is a first-class surface" | PLAN says reduced motion is a designed surface, ships with v1, and accessibility is a launch blocker | Plan-derived |
| "Privacy-respecting observability" | PLAN observability and privacy boundary sections | Plan-derived |
| "Rich aliveness inside strict performance budgets" | PLAN performance budgets tied to first-frame and ambient sessions | Plan-derived paraphrase |

Rubric-side vocabulary such as "intent fidelity," "multi-layer recovery," "weight-3," and "feature-level fidelity" does not appear. The word "canonical" appears, but it is a plan term, not a leakage sign here.

## Heading Mirror Check

| Reconstruction heading | Closest source | Assessment |
|---|---|---|
| "System-level intent" | Phase-2A expected reconstruction shape; not a gold title | Not suspicious |
| "Per-feature whys" | Phase-2A expected reconstruction shape; not a gold title | Not suspicious |
| "1. Scope" through "12. Risks" | PLAN section headings | Mirrors PLAN, not GOLD_WHYS |

The headings do not echo gold-list titles such as "System-level whys" or individual S/F labels. They mirror the plan's implementation-plan structure.

## 1:1 Mapping Suspect Check

**Result: PASS.** The reconstruction does not provide a neat item for every S1-S9 or F1-F40 target, does not use those IDs, and does not follow the gold order. It instead walks the plan from Scope through Risks and includes many non-gold plan features alongside explicit NOT RECOVERABLE statements. That shape is consistent with plan derivation.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The drift model also says 'Neglect produces zero delta; traits never decrease,' so absence is not punished." | PLAN §5.2 Monotonic rule: deltas are always >= 0; neglect produces zero delta; traits never decrease | Supported |
| "The stated why is to avoid a sync problem: 'there is no client state to merge.'" | PLAN §6.1: server holds the only canonical copy; clients are thin renderers; no sync problem because no client state to merge | Supported |
| "If audio is unavailable, silence is preferred 'to canned audio,' while captions keep the aviary functional." | PLAN §8.4 WebAudio fallback: audio silenced, captions enabled, silence preferable to canned audio | Supported |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold ID leakage, no rubric vocabulary leakage, no gold-list heading mirror, and no 1:1 gold-target mapping. The few strong phrases are traceable to the PLAN, and the frequent NOT RECOVERABLE entries argue against contamination rather than for it.
