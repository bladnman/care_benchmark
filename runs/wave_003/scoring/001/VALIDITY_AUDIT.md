# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS.

I found no gold why IDs or target IDs in the frozen reconstruction. The reconstruction does not use tokens such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. It also does not introduce an external scoring taxonomy. The only numbered language is ordinary plan-derived material such as `1-2 seconds`, `60-second`, and endpoint paths.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `browser-based, contemplative virtual aviary where idle attention is the primary mode of interaction` | PLAN.md:6 exact wording | plan-derived |
| `observational viewport` | PLAN.md:40 exact wording | plan-derived |
| `tranquil ambient sky field` / `never a spinner` | PLAN.md:141 exact wording | plan-derived |
| `monotonically unlocked strictly by aviary age` | PLAN.md:11 exact wording | plan-derived |
| `NEVER decay or drop on neglect` | PLAN.md:94 exact wording | plan-derived |
| `idempotent append-only log` | PLAN.md:128 exact wording | plan-derived |
| `internal synthetic UUIDv4` | PLAN.md:60 exact wording | plan-derived |
| `not an "animations off" degradation` | PLAN.md:146 exact wording | plan-derived |
| `Capitalized, direct, clear, no affected warmth` | PLAN.md:202 exact wording | plan-derived |
| `scene-centered` | Inference from PLAN.md:7,136 and reconstruction's plan-structure summary | minor paraphrase, not scorer vocabulary |

Suspicious scorer-side phrases checked: `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, and `intent fidelity`. None appear in the frozen reconstruction.

## Heading Mirror

Required phase-2A headings are present: `## System-level intent` and `## Per-feature whys`. These are mandated by the reconstructor prompt and are not gold leakage.

The `###` headings mirror the PLAN structure, not the gold list:

| Reconstruction heading | Mirrors PLAN? | Mirrors GOLD_WHYS? | Assessment |
|---|---|---|---|
| `Scope and Architectural Non-Negotiables` | Yes, PLAN section 1 | No | plan-derived |
| `System Architecture & Topology` | Yes, PLAN section 2 | No | plan-derived |
| `Data Models & Schema Design` | Yes, PLAN section 3 | No | plan-derived |
| `Simulation Engine Design` | Yes, PLAN section 4 | No | plan-derived |
| `API Surface & Client-Server Sync Model` | Yes, PLAN section 5 | No | plan-derived |
| `Frontend Rendering & Audio Pipelines` | Yes, PLAN section 6 | No | plan-derived |
| `Interaction Protocols & UR Specifications` | Yes, PLAN section 7 | No | plan-derived |
| `Accessibility & Voice Separation` | Yes, PLAN section 8 | No | plan-derived |
| `Performance Budgets & Observability` | Yes, PLAN section 9 | No | plan-derived |
| `Rollout Strategy, Population Ramp & Risks` | Yes, PLAN section 10 | No | plan-derived |

## 1:1 Mapping Suspect

Verdict for this check: PASS.

The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve the gold order, and does not produce exactly 49 target rows. It follows the candidate PLAN's section order and feature grouping, including plan-specific misspellings such as `linteraction_eventsa`, `visit_logsa`, and `gest_token`. That argues strongly for plan-derived reconstruction rather than a hidden gold-list mapping.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| `Server authority protects consistency and prevents overwrite behavior.` | PLAN.md:19 names the authoritative server-side tick; PLAN.md:126-129 says the server is sole author and clients never send absolute state. | supported |
| `Privacy is a hard boundary around identity, telemetry, and social viewing.` | PLAN.md:59-62 define UUID/email and telemetry boundaries; PLAN.md:20 frames quiet read-only guest links/private visit log. | supported |
| `Accessibility is a designed equivalent surface, not a degraded fallback.` | PLAN.md:21 includes narration/reduced motion/captions/keyboard/contrast; PLAN.md:144-149 says reduced motion is not an animations-off degradation and keeps audio/captions/mood/notebook active. | supported |

## Verdict

PASS. The frozen reconstruction reads as a faithful plan-derived artifact. It follows the PLAN's structure, repeats and paraphrases PLAN vocabulary, includes honest `NOT RECOVERABLE FROM PLAN` entries, and shows no gold-ID, rubric-vocabulary, heading-mirror, or 1:1 target-list contamination signatures.
