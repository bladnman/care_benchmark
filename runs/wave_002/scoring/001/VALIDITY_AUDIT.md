# VALIDITY_AUDIT - CARE run 001

## Verdict

**PASS.** The frozen reconstruction reads as derived from PLAN.md. It uses the phase-2A structure and mirrors the plan's own section order, but it does not leak gold IDs, rubric scoring vocabulary, or a neat S1-S9/F1-F40 target map.

## 1. ID Leakage Check

No gold IDs were found in the reconstruction: no S1-S9 labels, no F1-F40 labels, no R-F identifiers, and no gold-list taxonomy. The only notable non-plan phrase is the phase-2A convention "NOT RECOVERABLE FROM PLAN", which appears on reconstruction lines 31, 41, 47, 109, 121, 275, 351, 451, and 455. That phrase is absent from PLAN.md, but it is a reconstruction-reporting marker rather than a gold why ID or PRD-specific taxonomy.

Cross-check: searching PLAN.md for the same gold-ID patterns found no corresponding leaked S/F IDs.

## 2. Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| continued between visits (line 3) | PLAN.md:7 uses the same phrase | Plan-derived |
| already-running scene (line 3) | PLAN.md:17 uses the same phrase | Plan-derived |
| server-authored canonical truth (line 5) | Exact phrase absent, but PLAN.md:13, :15, :56, :61, and :179 repeatedly establish server-only canonical authority and server-authored tracks/deltas | Plan-derived abstraction |
| immutable UUID and call identity (line 7) | PLAN.md:14 uses the same phrase | Plan-derived |
| nonnegative deltas (line 9) | PLAN.md:13 uses the same phrase | Plan-derived |
| naturalist prose (line 11) | PLAN.md:81 and :341 use the phrase | Plan-derived |
| procedurally synthesized (line 13) | PLAN.md:18 uses the phrase | Plan-derived |
| Accessibility ships with the primary experience (line 15) | PLAN.md:9 uses the same sentence | Plan-derived |
| correct ordinary browser behavior (line 19) | PLAN.md:223 uses the same phrase | Plan-derived |
| historical transparency (line 21) | PLAN.md:391 uses the phrase | Plan-derived |

Rubric/gold-side vocabulary such as multi-layer recovery, feature-level fidelity, intent fidelity, weight-3, and load-bearing was not found in RECONSTRUCTION.md.

## 3. Heading Mirror Check

The top-level headings System-level intent and Per-feature whys match the required reconstruction structure. The remaining reconstruction headings mirror PLAN.md section titles and order: Release scope and governing invariants, Decisions where the supplied specifications leave room or conflict, Architecture and boundaries, Data model and retention, API contracts, Server simulation and exact update semantics, Presence, interactions and multi-device sync, Frontend scene and interaction surfaces, Procedural audio and captions, Accessible experience and naturalist writing, Notebook, adoption and account experience, Visits, revocation and privacy boundaries, Performance budgets and observability, and Verification strategy, delivery, rollout, and risks.

They do not mirror GOLD_WHYS.md headings such as S1 - feels-alive-not-robotic, F1 - presence-definition, or the gold file-group heading sequence. This is not a contamination signal.

## 4. 1:1 Mapping Suspect Check

The reconstruction does not create one neat item per gold target in gold order. Instead, it walks the plan's implementation outline and includes many non-gold implementation details such as endpoint contracts, data records, cache bounds, audio worklet behavior, backup-key destruction, and rollout gates. The overlap with gold feature domains is expected because both documents describe the same product, but there is no S1-S9/F1-F40 ordered mapping and no gold IDs.

Verdict for this check: not suspect.

## 5. Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| The plan repeatedly centers an aviary that "feels as though it has continued between visits"... (line 3) | PLAN.md:7 says the product feels continued between visits; :17 says first frame is already-running; :56 processes accounts without clients; :13 says absence cannot lose traits/birds/names/drift; :510 says launch preserves birds the user has come to know | Supported |
| Presence requires a visible document, focused window, and recent pointer/key activity... (line 19) | PLAN.md:16 states the exact three conditions; :219-223 gives the implementation and bounded-credit rationale; :225 says presence is private input and not a duration/streak/calendar | Supported |
| Structural privacy enforcement: simulation storage, identity decryption, logs, aggregate metrics... (line 471) | PLAN.md:395-402 lists those structural controls; :429-433 defines allowed aggregate metrics and forbidden per-account/per-bird telemetry | Supported |

No spot-checked sentence required gold-list knowledge to derive.

## 6. Verdict

**PASS.** The reconstruction is broad and strongly organized, but its organization follows PLAN.md rather than the gold list. It uses no S/F IDs, no scoring vocabulary, and no unsupported 1:1 gold map. Minor abstract phrasing, such as server-authored canonical truth, is supported by repeated plan language and does not rise to a flag.
