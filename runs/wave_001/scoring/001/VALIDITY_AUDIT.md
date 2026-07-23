# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict for this check: PASS. A mechanical scan of the frozen reconstruction found no tokens matching gold IDs such as `F1`, `S1`, `R-F01`, or `R-S01`. Since no hits were present, there were no reconstruction IDs to cross-check against the plan.

## Vocabulary Check

Verdict for this check: PASS. No scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity` appeared in the frozen reconstruction.

Sampled plan-derived phrases:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "slow, ambient relationship-building" | Executive summary uses the same phrase. | plan-derived |
| "Server-Authored Canonical State" | Non-negotiable invariant 1 uses the same heading. | plan-derived |
| "Strict Conjunction Presence Model" | Non-negotiable invariant 2 uses the same heading. | plan-derived |
| "Monotonic Positive-Only Personality Drift" | Non-negotiable invariant 3 uses the same heading. | plan-derived |
| "No Announcement UI" | Non-negotiable invariant 5 uses the same phrase. | plan-derived |
| "lowercase, present-tense naturalist" | Invariant 5, notebook, and narration sections use this language. | plan-derived |
| "Procedural WebAudio Synthesis" | Invariant 4 and audio pipeline sections use this phrase. | plan-derived |
| "Telemetry & Privacy Wall" / privacy wall | PLAN section 10.3 has the same boundary framing. | plan-derived |

## Heading Mirror

Verdict for this check: PASS. The `## System-level intent` and `## Per-feature whys` headings are required by the phase-2A reconstruction prompt. The `###` headings under per-feature whys mirror the PLAN's numbered sections: Executive Summary, Scope, Architecture, Data Schemas, API Surface, Simulation Engine, Sync, Frontend, WebAudio, Accessibility, Performance, Rollout, and Risk Matrix. They do not mirror the held-out gold headings or S1-S9 / F1-F40 order.

## 1:1 Mapping Suspect

Verdict for this check: PASS. The reconstruction has 10 system-level bullets and then plan-section groupings, not 9 system-level entries followed by 40 feature-level entries. It does not enumerate S1-S9 or F1-F40, and it includes plan-local items such as API endpoints, Redis cache, SSE propagation, release sequence, and risk matrix mitigations. That shape is plan-derived rather than gold-list-derived.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan's care philosophy is `Monotonic Positive-Only Personality Drift`: traits move toward expressiveness and `never` down." | PLAN invariant 3 and §5.2 state the monotonic drift invariant and no-subtraction rule. | grounded |
| "Event ingestion is a write-ahead log for the tick rather than a place for client state mutation." | PLAN §2.2 says event ingestion prevents client-driven state corruption by acting as a write-ahead log. | grounded |
| "The rationale is to allow operational metrics while forbidding export or logging of per-bird vectors, mood histories, individual presence duration, notebook contents, emails, and hashes." | PLAN §10.3 lists permitted operational telemetry and strictly forbidden telemetry in those categories. | grounded |

## Verdict

PASS. The reconstruction reads as a plan-derived summary: it mirrors the assigned PLAN's structure, repeats PLAN vocabulary, contains no gold IDs, contains no scorer/rubric vocabulary, and does not map neatly to the held-out gold why list. The run's scores do not need contamination discounting.
