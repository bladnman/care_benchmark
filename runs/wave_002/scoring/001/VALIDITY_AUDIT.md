# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result: PASS.** I searched the frozen reconstruction for gold/rubric identifiers and adjacent forms, including `S1`-`S9`, `F1`-`F40`, `R-F*`, `canonical #*`, `feature-level fidelity`, `multi-layer`, `weight-2`, `weight-3`, `intent fidelity`, `gold why`, and `rubric`. There were no hits in `runs/wave_002/reconstructions/001/RECONSTRUCTION.md`, so there are no offending IDs to cross-reference against the PLAN.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "already be alive on first sight" | PLAN sec. 0 says the first frame is already in motion and never spinner-then-fade. | Plan-derived. |
| "nothing on any surface announces the user" | PLAN sec. 0 uses the same rule and sec. 1.3 adds no announcement primitives. | Plan-derived. |
| "The boundary is the snapshot" | Exact PLAN sec. 2.4 phrase. | Plan-derived. |
| "watching without moving is the product" | Exact PLAN sec. 15.5 phrase. | Plan-derived. |
| "no call is ever repeated exactly" | Exact PLAN sec. 5.9 phrase. | Plan-derived. |
| "no credential to the sim database" | Exact PLAN sec. 10.5 phrase. | Plan-derived. |
| "designed surface, not a later pass" | PLAN secs. 7.8 and 9.7 support this accessibility framing. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | A reconstruction-scoping marker, used sparingly for plan gaps. | Not gold/rubric leakage. |

I did not find rubric-side vocabulary such as `multi-layer recovery`, `feature-level fidelity`, `weight-3`, `gold why`, or `intent fidelity` in the reconstruction.

## Heading Mirror

Reconstruction headings:

| Heading | Comparison to gold-list headings | Assessment |
|---|---|---|
| `## System-level intent` | Expected phase-2A output shape; not a gold title. | Not suspect. |
| `## Per-feature whys` | Expected phase-2A output shape; not a gold title. | Not suspect. |
| `### How to read this plan and scope` | Mirrors PLAN sec. 0 and sec. 1, not GOLD_WHYS. | Plan-derived. |
| `### Architecture, data model, and API surface` | Mirrors PLAN secs. 2-4. | Plan-derived. |
| `### Simulation engine` | Mirrors PLAN sec. 5. | Plan-derived. |
| `### Sync model and frontend rendering` | Mirrors PLAN secs. 6-7. | Plan-derived. |
| `### Audio pipeline` | Mirrors PLAN sec. 8. | Plan-derived. |
| `### Accessibility, performance, privacy, rollout, and operations` | Mirrors PLAN secs. 9-13. | Plan-derived. |

No heading echoes a GOLD_WHYS feature group or ID list closely enough to flag.

## 1:1 Mapping Suspect

**Result: PASS.** The reconstruction does not contain a neat S1-S9/F1-F40 sequence. Its system section has 14 bullets rather than 9, and its per-feature section follows the PLAN's own sections and includes many non-gold implementation details such as edge hints, call-plan horizon, rate limits, WebGL rendering, operational runbooks, and feature flags. It also marks several entries `NOT RECOVERABLE FROM PLAN`, which is inconsistent with a gold-list mirror.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Presence means watching, honestly counted."
   PLAN support: sec. 5.3 says presence is "the user is watching," not devices being open; sec. 15.5 says watching without moving is the product; sec. 1.3 requires visible/focused/active e2e tests.
   Assessment: grounded.

2. Reconstruction sentence: "The aviary is deterministic but not canned."
   PLAN support: sec. 2.6 defines deterministic seeds and named RNG purposes; sec. 5.9 says per-call seed jitter means no call repeats exactly; sec. 5.11 uses noise and seed variation for greetings.
   Assessment: grounded.

3. Reconstruction sentence: "Privacy boundaries are structural."
   PLAN support: sec. 1.3 defines the telemetry allowlist; sec. 10.5 says the analytics warehouse has no sim database credential; sec. 10.7 enforces logger allowlists and dependency boundaries.
   Assessment: grounded.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as plan-derived: it uses PLAN section groupings and phrases, lacks gold IDs and rubric vocabulary, includes plan-specific implementation details, and does not mirror the gold target list in order or cardinality.
