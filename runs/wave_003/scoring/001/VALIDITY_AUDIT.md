# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this section: PASS.

A bounded search of `RECONSTRUCTION.md` found no gold why IDs such as `S1`-`S9`, `F1`-`F40`, or `R-Fxx`. The only hit from the combined leakage/vocabulary search was `load-bearing`, in the sentence about "one writer of personality". That phrase also appears in `PLAN.md` in the service-shape section, so it is not leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Finding |
|---|---|---|
| `load-bearing rule` | PLAN service shape: "load-bearing rule is one writer of personality" | supported |
| `one writer of personality` | PLAN service shape and sync sections | supported |
| `structurally impossible` | PLAN sync model says conflict on personality is structurally impossible | supported |
| `watching without moving is the product` | PLAN D2 says the same phrase | supported |
| `tab open overnight inflation` | PLAN presence decisions discuss open-tab inflation | supported |
| `No account_id, no bird ids, no event kinds` | PLAN RUM section uses the same bounded telemetry exclusion | supported |
| `No badge, no push` | PLAN visit-log route says the same | supported |
| `designed renderer, not animation: none` | PLAN reduced-motion section says the same | supported |
| `No recorded-audio path. Unconditional.` | PLAN WebAudio fallback says the same | supported |
| `could reconstruct a relationship` | PLAN observability exclusions use this phrase | supported |

No rubric-side terms such as `intent fidelity`, `feature-level fidelity`, `weight-3`, or `multi-layer recovery` appear in the reconstruction.

## Heading Mirror Check

`RECONSTRUCTION.md` headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Scope and decisions`
- `### 2. Architecture, data model, and API surface`
- `### 3. Simulation engine and sync model`
- `### 4. Frontend rendering pipeline`
- `### 5. Audio pipeline`
- `### 6. Accessibility surfaces`
- `### 7. Performance, observability, rollout, testing, and voice`

These headings mirror the PLAN's implementation sections, not the gold-list section titles. They do not enumerate S1-S9 or F1-F40 and do not use gold section labels.

## 1:1 Mapping Suspect Check

Verdict for this section: PASS.

The reconstruction does not give every gold target a neat corresponding item. It has 12 system-level principles rather than 9, and its per-feature section follows PLAN sections with many non-gold implementation items mixed in. The order is PLAN-derived rather than a clean S1-S9 / F1-F40 sequence.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Finding |
|---|---|---|
| `Privacy is a structural boundary, not just policy copy.` | PLAN 2.1 hard pipeline split, account-db/sim-db separation, and PLAN 10.3/10.4 telemetry exclusions | plan-derived |
| `Reduced motion is a designed renderer, not animation: none.` | PLAN 7.5 uses the same phrase and details crossfades, no leaf drift, unchanged audio/drift | plan-derived |
| `Sync correctness is framed around making conflict on personality structurally impossible.` | PLAN 6.1 says conflict is structurally impossible if only the tick writes vectors | plan-derived |

## Verdict

PASS.

The reconstruction reads as plan-derived. I found no gold ID leakage, no gold-heading mirror, no 1:1 mapping to the gold list, and no unsupported rubric vocabulary. The few load-bearing-sounding phrases checked against the PLAN were present there already.
