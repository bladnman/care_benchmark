# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no gold-ID leakage found. I searched the frozen reconstruction for gold-style IDs and rubric labels such as `S1`, `F1`, `F40`, `R-F01`, `weight-3`, `feature-level`, `multi-layer`, and `intent fidelity`. The only hit was the phrase `load-bearing` in RECONSTRUCTION.md line 103: "The plan calls this "load-bearing" for "monotonic toward expressive"...". The same phrase appears in PLAN.md line 223, so this is plan-derived, not leakage. No F/S/R IDs appear in the reconstruction.

## Vocabulary Check

Sampled load-bearing phrases against the plan:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "architectural invariant" | PLAN opening uses "architectural invariants" | plan-derived |
| "dominant drift input" | PLAN opening and §5/§6 use this wording | plan-derived |
| "ongoing life" | PLAN §2 says server sends current state of ongoing life | plan-derived |
| "not a kill switch" | PLAN §7 reduced-motion heading says this | plan-derived |
| "calmer register of the same aviary" | PLAN §7 uses this phrase | plan-derived |
| "aggregate-only" | PLAN §10/§11 repeatedly uses this | plan-derived |
| "load-bearing" | PLAN §3 modeling note uses it | plan-derived |
| "not ARIA-label automation" | PLAN §9 uses this phrase | plan-derived |
| "Tamagotchi-by-clicking" | PLAN §13 uses this phrase | plan-derived |

No suspicious rubric-only vocabulary appears freely.

## Heading Mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then numbered headings mirroring the candidate PLAN sections: Scope, Architecture, Data model, API surface, Simulation engine design, Sync model & presence, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Performance budgets & observability, Privacy & data boundary, Rollout, Risks, and Open calibration items. These mirror the PLAN structure, not GOLD_WHYS section titles or S1-F40 ordering.

## 1:1 Mapping Suspect

No near-1:1 gold mapping found. The reconstruction has 12 system-level bullets, not 9 S-whys, and its per-feature section follows the plan's fourteen numbered sections rather than the 40 gold feature whys. It includes many non-gold implementation bullets and several `NOT RECOVERABLE FROM PLAN` calls, which is consistent with plan-derived reconstruction rather than a gold-list skeleton.

## Plan-Derivation Spot Check

1. Reconstruction: "The aviary is ongoing life, not a session that starts when the tab opens." PLAN support: §2 render boundary says the server sends current phase/current state of ongoing life, §7 boots from an inlined snapshot mid-motion, and §5 tick runs without connected clients.

2. Reconstruction: "Settle and tab-close equivalence: Both are terminal, neither penalized, and there is no 'you didn't settle' surface." PLAN support: §6 states settle and tab-close are terminal and equivalent at the engine level, neither penalized, with no such surface.

3. Reconstruction: "Reduced-motion rendering register: Cross-fades, slowed color shifts, retained calls/captions, and ongoing drift make it 'a calmer register of the same aviary.'" PLAN support: §7 says reduced motion replaces micro-motion with cross-fades, keeps calls/captions/drift/notebook, and calls it a calmer register.

All three articulate sentences are directly supported by PLAN passages.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs leak, headings mirror the plan rather than the gold list, vocabulary is traceable to the plan, and the structure is not a neat S1-S9/F1-F40 mapping. The single rubric-sounding phrase found, `load-bearing`, is present in the plan itself.
