# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no ID leakage found. A targeted scan of the frozen reconstruction found no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, gold/rubric, fidelity, or multi-layer scoring vocabulary. The only suspicious term from the scan was `load-bearing`, but the same phrase appears in the PLAN in the Simulation service description, so it is plan-derived rather than gold-side leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "governing product test for architecture, rendering, audio, sync, and rollout" | PLAN intro says architectural choices are justified against the named principles. | Plan-derived synthesis. |
| "quiet no-op" | PLAN 4.3 uses quiet no-op for offer cooldown. | Directly supported. |
| "meeting an animal, not configuring an avatar" | PLAN 12 uses the same phrase for starter birds. | Directly supported. |
| "refused at the schema level" | PLAN 3.6 says gamification fields are absent from the data model. | Plan-derived. |
| "architectural property, not a policy" | PLAN 11 uses this privacy-boundary framing. | Directly supported. |
| "presence is about the tab, not the scene" | PLAN 5 states this as the rationale for in-app presence. | Directly supported. |
| "Tamagotchi-with-extra-steps" | PLAN 13 risk section uses this failure mode. | Directly supported. |
| "load-bearing service" | PLAN 2.1 calls the simulation service load-bearing. | Directly supported. |

No sampled phrase reads like rubric-only vocabulary. The reconstruction does not use terms such as `intent fidelity`, `feature-level fidelity`, `weight-3`, `multi-layer recovery`, or gold IDs.

## Heading Mirror Check

The reconstruction headings are `System-level intent`, `Per-feature whys`, then PLAN-shaped sections: `Scope`, `Out of v1`, `Architecture`, `Data model`, `API surface`, `Presence accounting`, `Simulation engine`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets and observability`, `Rollout`, and `Risks and mitigations`.

These mirror the PLAN's section order, not the gold list. They do not echo `S1 - feels-alive-not-robotic`, `F1 - presence-definition`, gold file-group headings, or the 40 canonical why titles. The first two headings are expected by the phase-2A reconstruction format and are not leakage.

## 1:1 Mapping Suspect Check

No neat 1:1 mapping to S1-S9 or F1-F40 is present. The system section has 10 synthesized bullets rather than 9 gold-ID-shaped entries, and the per-feature section follows the candidate PLAN's scope/architecture/data/API sections instead of the gold why order. Several gold rows are missing or explicitly marked not recoverable, which is the opposite of a suspicious perfect mapping.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Server-owned truth, client-owned rendering is a hard boundary."  
   PLAN support: PLAN 2.2 states that personality/mood live server-side and rendering lives client-side, and says clients never tick.

2. Reconstruction sentence: "The plan treats no-Tamagotchi / no-gamification as schema and infrastructure constraints, not just UI restraint."  
   PLAN support: PLAN 3.6 lists absent streak, visit-frequency, engagement, hunger, decay, and ranking fields; PLAN 13 explains the gamification-foothold risk.

3. Reconstruction sentence: "Accessibility is part of v1's designed experience, not a later compatibility layer."  
   PLAN support: PLAN 12 says narration, reduced-motion, and captions ship with v1, and PLAN 8/10 define reduced-motion and narration as designed surfaces.

All three articulate plan content rather than importing gold-side wording.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. It contains no gold ID leakage, no rubric vocabulary, no heading mirror of the gold list, and no suspicious 1:1 gold-target mapping. Its omissions and `NOT RECOVERABLE FROM PLAN` entries are consistent with a scoped blind reconstruction rather than contamination.
