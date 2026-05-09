# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: **no ID leakage found**.

I searched the frozen reconstruction for gold-style IDs (`S1`-`S9`, `F1`-`F40`, and `R-F...`) and found no hits. The reconstruction uses plan-native feature names and section groupings rather than gold IDs. There was therefore nothing to cross-reference against `PLAN.md` for an offending ID.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "The client is a dumb renderer of a smart server" | PLAN.md:30 uses the same boundary sentence. | Plan-derived. |
| "single source of truth" | PLAN.md:61. | Plan-derived. |
| "quiet field background" | PLAN.md:70. | Plan-derived. |
| "entry illusion" | PLAN.md:101. | Plan-derived. |
| "alive feeling" | PLAN.md:95. | Plan-derived. |
| "strictly monotonic toward expressive" | PLAN.md:55. | Plan-derived. |
| "slow, naturalist prose updates" | PLAN.md:79. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Not in PLAN, but expected phase-2A marker language rather than gold-side content. | Not a contamination hit. |

No rubric-side vocabulary such as "multi-layer", "feature-level fidelity", "weight-3", "gold", "rubric", or "intent fidelity" appears in the reconstruction.

## Heading Mirror Check

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- Plan-section mirrors: `Scope`, `Explicit Non-Goals`, `Architecture`, `Data Model`, `API Surface`, `Simulation Engine Design`, `Sync Model`, `Frontend Rendering Pipeline`, `Audio Pipeline`, `Accessibility Surfaces`, `Performance Budgets and Observability`, `Rollout`, `Risks`.

The two top headings match the expected reconstruction format. The `###` headings mirror the plan's own section headings rather than gold-list sections. They do not echo the gold S1-S9 or F1-F40 list.

## 1:1 Mapping Suspect Check

Verdict: **not suspect**.

The reconstruction does not enumerate S1-S9 or F1-F40 and does not proceed in gold-list order. It has 8 system bullets, then a plan-section walk that includes many non-gold features and operational items. This shape matches the plan's structure, not the gold taxonomy.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Architecture states the boundary directly: 'The client is a dumb renderer of a smart server.'" | PLAN.md:30: "The client is a dumb renderer of a smart server." | Directly plan-derived. |
| "The plan says loading should start from a 'quiet field background' with motion already in progress, avoiding load states that break the 'entry illusion.'" | PLAN.md:70 has quiet field/motion in progress; PLAN.md:101 says load states can break the entry illusion. | Plan-derived synthesis across two plan lines. |
| "Ramping is based on 'weeks/months, not engagement' and unlocks up to five additional birds without using engagement mechanics." | PLAN.md:94: aviary age measured in weeks/months, not engagement, unlocks up to 5 additional birds. | Directly plan-derived. |

## Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, no gold-order mapping, no rubric scoring vocabulary, and the strongest phrases trace directly to `PLAN.md`. The only non-plan phrase pattern is the expected `NOT RECOVERABLE FROM PLAN` marker, which supports validity rather than contamination.
