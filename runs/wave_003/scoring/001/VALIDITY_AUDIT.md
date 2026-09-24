# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage detected.

The frozen reconstruction does not use gold why IDs such as S1-S9, F1-F40, R-F*, or R-S*. Its headings and bullets use the PLAN's section names and product vocabulary rather than scorer-side identifiers. Numeric strings that appear, such as "30 days," "five seconds," "seven," and "500 ms," are product constants already present in the PLAN.

## Vocabulary Check

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "observational and quiet" | PLAN sec.1 uses the same phrase. | Plan-derived. |
| "server is the only writer" | PLAN sec.1/sec.2 states the server/simulation worker is the only writer. | Plan-derived. |
| "quiet watching counts but abandoned tabs do not" | PLAN sec.11 uses this calibration wording. | Plan-derived. |
| "hidden-vector invariant" | PLAN sec.3 uses this phrase around export conflict resolution. | Plan-derived. |
| "launch requirements, not a later phase" | PLAN sec.10 uses this sentence for accessibility. | Plan-derived. |
| "release gates" | PLAN sec.9 says budgets are release gates. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Required reconstructor marker, not gold/rubric vocabulary. | Expected. |

I found no suspicious use of scorer-side terms such as "gold," "rubric," "weight-3," "feature-level fidelity," "system-level fidelity," or "multi-layer recovery" in the reconstruction.

## Heading Mirror

Top-level headings are exactly the required reconstruction headings: `System-level intent` and `Per-feature whys`. Subheadings mirror the PLAN's own implementation sections: `Product scope and invariants`, `Architecture and ownership`, `Data model`, `API surface and authorization`, `Simulation engine`, `Client render and interaction pipeline`, `Audio and captions`, `Accessibility, account surfaces, and voice`, `Performance, observability, and privacy`, `Rollout and release sequence`, and `Main risks and mitigations`.

These headings do not mirror the gold-list sections. They are plan-derived and not suspicious.

## 1:1 Mapping Suspect

Verdict: not suspect.

The reconstruction does not create neat S1-S9 or F1-F40 rows. It is much longer than the 49 gold-whys surface and is grouped by the PLAN's implementation sections, with many bullets for non-gold features and several explicit `NOT RECOVERABLE FROM PLAN` calls. The order follows the PLAN, not the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Presence should count quiet watching but not abandoned tabs." | PLAN sec.11: calibrate heartbeat validity so quiet watching counts but abandoned tabs do not. | Supported. |
| "Personality values are categorically hidden from interactive surfaces and export contracts." | PLAN sec.1 says keep values off interactive surfaces; sec.3 follows the stronger hidden-vector invariant for export. | Supported. |
| "Accessibility is part of v1, using the same facts as the scene." | PLAN sec.8 generates prose from snapshot/event facts; sec.10 says these are launch requirements, not a later phase. | Supported. |

## Verdict: PASS

The reconstruction reads as plan-derived. It uses the PLAN's section structure and vocabulary, contains no gold ID leakage or rubric-side vocabulary, and includes honest `NOT RECOVERABLE FROM PLAN` markers where the PLAN gave a rule without a rationale. Scores do not need contamination discounting.
