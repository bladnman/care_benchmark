# VALIDITY_AUDIT — CARE run 001

## Verdict

**PASS.** No significant contamination signatures found. The reconstruction follows the PLAN's section order and vocabulary, uses no gold IDs, and does not form a neat S1-S9/F1-F40 mapping.

## ID Leakage

I searched the frozen reconstruction for gold IDs and rubric-like IDs (`S1`-`S9`, `F1`-`F40`, `R-F*`). There were no gold-ID hits. The only notable uppercase diagnostic string was `NOT RECOVERABLE FROM PLAN`, which is expected phase-2A reconstruction vocabulary rather than a gold-list identifier.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "load-bearing asymmetry test" | yes | PLAN uses this exact phrase in the drift section. |
| "server-canonical relationship" | yes | PLAN repeatedly uses server-canonical framing. |
| "canonical state vs presentation" | yes | PLAN names this boundary directly. |
| "matter-of-fact tone" | yes | PLAN uses this for system surfaces. |
| "different rendering of the same aviary" | yes | PLAN uses this for reduced motion. |
| "affective-perf threshold" | yes | PLAN names time-to-first-bird this way. |
| "NOT RECOVERABLE FROM PLAN" | no | Reconstruction-side judgment phrase; not gold-specific. |
| "multi-layer", "feature-level fidelity", "weight-3" | no hits | Rubric-side vocabulary did not appear. |

## Heading Mirror

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, and numbered plan-section headings such as `### 1. Scope`, `### 2. Architecture`, through `### 15. Open questions`. These mirror the PLAN's organization rather than `GOLD_WHYS.md` headings. They do not echo gold why titles like `feels-alive-not-robotic`, `notice-never-announce`, or feature IDs `F1`-`F40`.

## 1:1 Mapping Suspect

No 1:1 gold mapping detected. The reconstruction lists many plan-derived items in section order and includes plan-only implementation details such as Solid.js, Redis, SSE, Postgres, feature flags, rollout phases, and open questions. It does not enumerate exactly 9 system whys plus 40 feature whys, and it does not follow the gold-list order.

## Plan-Derivation Spot Check

1. Reconstruction: "anything that affects the canonical relationship the user has with their birds" runs server-side.
   PLAN support: §2.2 contains the same sentence and contrasts canonical state with presentation.

2. Reconstruction: refusing engagement features at the data layer is more durable than refusing them in UI.
   PLAN support: §1.2 says refusing features at the data layer is more durable than refusing them at the UI layer.

3. Reconstruction: reduced motion is "a different rendering of the same aviary" and should not feel broken.
   PLAN support: §7.6 and §9.2 describe reduced motion as a different rendering, calmer and slower, not a stripped fallback.

## Final Rationale

The reconstruction contains strong PLAN-derived phrasing, including some exact plan phrases, but that is expected because phase 2A read the PLAN. I found no gold ID leakage, no rubric taxonomy leakage, no heading mirror to the gold list, and no suspicious one-to-one S/F mapping. Verdict: **PASS**.
