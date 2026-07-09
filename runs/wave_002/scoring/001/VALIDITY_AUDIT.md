# VALIDITY_AUDIT - run 001

## 1. ID leakage check

Verdict: no ID leakage found. A mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because there were no hits, no PLAN cross-reference leakage hit is possible.

## 2. Vocabulary check

A search for scorer-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, and `intent fidelity` returned no hits.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "primary interaction is attentive observation" | PLAN section 1 uses the exact phrase. | Plan-derived. |
| "Continuity rather than performance" | PLAN section 1 uses the exact heading. | Plan-derived. |
| "Noticing rather than announcing" | PLAN section 1 uses the exact heading. | Plan-derived. |
| "A private relationship rather than an engagement system" | PLAN section 1 uses the exact heading. | Plan-derived. |
| "server owns canonical simulation state" | PLAN opening paragraph uses the phrase. | Plan-derived. |
| "snapshot projection" | PLAN section 2 defines snapshot projection. | Plan-derived. |
| "not a hidden list of engine values" | PLAN accessibility section uses the exact phrase. | Plan-derived. |
| "not an animation kill switch" | PLAN reduced-motion bullet uses the exact phrase. | Plan-derived. |
| "without turning the product into a dashboard" | PLAN closing sentence uses the exact phrase. | Plan-derived. |

## 3. Heading mirror check

The reconstruction headings mirror the PLAN structure, not the gold list. The required top-level headings `## System-level intent` and `## Per-feature whys` come from the reconstruction prompt. The numbered `###` headings then follow PLAN sections: delivery intent, target architecture, domain model, public API, simulation engine, sync/client session, frontend scene, interactions, audio/captions, accessibility/content, performance/security/observability, and delivery sequence.

No heading is an exact or near-exact mirror of a held-out gold section such as "System-level whys" or "Feature-level whys." The heading mirror pattern is therefore expected plan derivation.

## 4. 1:1 mapping suspect check

Verdict: not suspect. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not present 49 neat target rows in gold order. It follows the plan's own sections and includes many more implementation items than the gold-why list. Several gold targets are merged into plan-shaped bullets, and some plan bullets are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with the phase-2A instructions.

## 5. Plan-derivation spot check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Qualified presence is evidence, not raw time spent." | PLAN defines qualified intervals as visible + focused + recent pointer/key activity and rejects background/unfocused/idle cases. | Grounded. |
| "Accessibility is a designed representation of the same canonical state." | PLAN says to treat accessible output as "a designed representation of the same canonical state, not a hidden list of engine values." | Grounded; near-exact plan wording. |
| "The plan is successful only if operational correctness and privacy are achieved without turning the product into a dashboard." | PLAN closing sentence uses this wording and adds that the server makes continuity real while the client makes it quiet, immediate, specific, and accessible. | Grounded; near-exact plan wording. |

## 6. Verdict

PASS. The reconstruction reads as plan-derived: it uses the plan's section order, headings, and vocabulary; it contains no gold IDs, no scorer/rubric terminology, and no neat 1:1 mapping to the hidden gold list. The few strong-sounding phrases are either exact plan phrases or directly supported by nearby plan passages.
