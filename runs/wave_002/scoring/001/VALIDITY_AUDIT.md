# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. Mechanical search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+` and `R-S[0-9]+` in the frozen reconstruction returned no hits. The reconstruction does not use gold IDs or the scorer taxonomy.

## Vocabulary Check

No scorer-side vocabulary was found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

Sampled plan-derived phrases:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "feels continuous across visits and devices" | PLAN §1 uses the same phrase. | Plan-derived. |
| "canonical server simulation" / "no client can overwrite personality" | PLAN §1 and §2 contain both ideas directly. | Plan-derived. |
| "lowercase, present-tense, particular observations" | PLAN §1 copy rule and §9 voice review. | Plan-derived. |
| "all accessibility surfaces" and "reduced-motion users" | PLAN §1 launch scope and §9 quality review. | Plan-derived. |
| "500ms first-bird target" | PLAN §11 performance budget. | Plan-derived. |
| "human recognition at seven birds" | PLAN §13 audio-risk response. | Plan-derived. |

The reconstruction occasionally uses concise evaluator-like summaries such as "quiet, non-engagement product posture," but those read as paraphrases of repeated plan language rather than rubric leakage.

## Heading Mirror

The reconstruction has only two `##` headings: `System-level intent` and `Per-feature whys`. These are required by the phase-2A prompt and do not mirror the gold-list section titles beyond the benchmark-required output format. There are no `###` headings and no copied gold headings.

## 1:1 Mapping Suspect

PASS. The reconstruction does not enumerate S1-S9 or F1-F40 and does not follow the gold order. It has 12 system-level principles and 167 plan-derived feature/implementation items, mirroring the plan's own sections rather than the gold taxonomy. Some items align with gold features because the plan is comprehensive, but the mapping is not a neat 49-item 1:1 sequence.

## Plan-Derivation Spot Check

1. Reconstruction: "The plan repeatedly separates durable truth from rendering." PLAN support: §2 says client work is presentation, the browser never chooses mood or applies trait deltas, and the simulation worker is the sole role authorized to update personality and canonical mood. Verdict: supported.

2. Reconstruction: "The notebook and observation memory support claims like 'first greeting change this week' while forbidding 'user visit streaks'." PLAN support: §3 observation memory supports first-greeting changes and never user visit streaks; §9 notebook candidates refer to birds, not attendance. Verdict: supported.

3. Reconstruction: "Performance must be honest and compatible with privacy." PLAN support: §11 defines the 500ms target end-to-end and says not to hide cold failures; §2 requires private/no-store snapshots and prohibits shared caching of private state. Verdict: supported.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it quotes and paraphrases the plan heavily, uses the required phase-2A headings, avoids gold IDs and rubric vocabulary, and is not organized as a gold-list mirror.
