# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. I checked the frozen reconstruction for gold-side identifiers and rubric-like IDs (`F1`, `F40`, `S1`, `S9`, `R-F01`, `gold`, `rubric`, `intent fidelity`, `feature-level fidelity`, `weight-3`). The reconstruction does not use gold IDs or external taxonomy. It uses ordinary plan-derived headings and bullets.

## Vocabulary Check

Sampled reconstruction phrases and plan support:

| Reconstruction phrase | PLAN support | Judgment |
|---|---|---|
| "felt-aliveness, naturalist voice, restraint" | PLAN intro uses the same phrase cluster. | plan-derived |
| "unreachable, not just discouraged" | PLAN intro says shortcuts are made unreachable, not discouraged. | plan-derived |
| "server is the only writer" | PLAN architecture repeats this as a hard boundary. | plan-derived |
| "affective spine" | PLAN audio says the audio pipeline is the affective spine. | plan-derived |
| "designed surface, not a checklist" | PLAN accessibility says accessibility is designed, not checklist. | plan-derived |
| "not a stripped fallback" | PLAN reduced-motion section uses this exact framing. | plan-derived |
| "does not have a dashboard" | PLAN rollout says the product's success does not have a dashboard. | plan-derived |
| "catastrophic, silent" | PLAN risk I labels personality-vector loss catastrophic and silent. | plan-derived |
| `NOT RECOVERABLE FROM PLAN` | This is reconstruction workflow language, not gold-side content; it appears only as an honest non-recovery marker. | acceptable |

No sampled phrase used rubric-only vocabulary such as "multi-layer recovery", "feature-level fidelity", or "weight-3".

## Heading Mirror Check

The reconstruction has only two markdown headings: `## System-level intent` and `## Per-feature whys`. These match the expected phase-2A reconstruction structure rather than the gold list. The bullet headings follow the PLAN's own order (`Scope`, `Architecture`, `Data stores`, `Simulation`, `Sync`, `Frontend`, `Audio`, `Accessibility`, `Performance`, `Rollout`, `Risks`, `Cross-cutting`) and do not mirror `GOLD_WHYS.md` S1-S9 or F1-F40 section titles.

## 1:1 Mapping Suspect Check

No neat 1:1 gold mapping was found. The reconstruction includes many plan-derived bullets far beyond the 49 gold whys and proceeds in PLAN order, not gold order. It does not enumerate S1-S9 or F1-F40. Several gold targets receive no corresponding reconstruction item (for example F10 and F39), while many non-gold plan details receive bullets, which argues against gold-list contamination.

## Plan-Derivation Spot Check

1. Reconstruction: "The audio section calls the audio pipeline the 'affective spine' and says 'if it sounds canned, the product reads as theater.'"
   PLAN support: section 8 says the audio pipeline is the affective spine and that canned sound makes the product read as theater.
   Judgment: grounded.

2. Reconstruction: "Telemetry exists for operational health, not relationship tracking."
   PLAN support: sections 10 and 11 say RUM is aggregate-only, per-bird data never reaches analytics, and the product monitors health rather than engagement.
   Judgment: grounded.

3. Reconstruction: "`bird_id` is never reused or reset because identity continuity protects the user's relationship with each bird."
   PLAN support: sync identity continuity states `bird_id` is never reused/reset/regenerated/swapped and risks describe vector loss as destroying the relationship.
   Judgment: grounded.

## Verdict: PASS

The reconstruction reads as plan-derived. It follows PLAN order, uses PLAN vocabulary, lacks gold IDs, lacks rubric scoring language, and does not provide neat gold-list coverage. The non-recoverable markers and missing gold targets are evidence against contamination rather than for it.
