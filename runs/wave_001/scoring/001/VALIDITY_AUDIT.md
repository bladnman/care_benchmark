# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Mechanical search of the frozen reconstruction for `F#`, `S#`, `R-F#`, and `R-S#` tokens returned no hits. The matching search of PLAN also returned no hits. Verdict for this check: no gold-ID leakage.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "entire value is affective" | PLAN section 0: "entire value is affective" | plan-derived |
| "quieter, not mistrustful" | PLAN section 0 and 7.4 use the same phrase | plan-derived |
| "load-bearing" | PLAN section 9.2: recognizability is the "load-bearing affordance" | plan-derived, not a leakage hit |
| "golden-vector tests" | PLAN sections 3 and 7.1 mention golden-vector tests | plan-derived |
| "product's immune system" | PLAN section 14.4 uses this phrase for the charm-guard suite | plan-derived |
| "quiet-field fallback" | PLAN sections 0, 8.2, and 15.4 use quiet-field/quiet field | plan-derived |
| "feature-level fidelity" | absent from reconstruction | no scorer vocabulary |
| "multi-layer recovery" | absent from reconstruction | no scorer vocabulary |
| "weight-3" | absent from reconstruction | no scorer vocabulary |

The only phrase that could look scorer-side in isolation is "load-bearing," but it appears in the PLAN itself. No suspicious rubric/gold vocabulary appears freely in RECONSTRUCTION.

## Heading Mirror

Reconstruction headings are:

| Heading | Comparison |
|---|---|
| `## System-level intent` | Required by the reconstruction prompt; not a gold-list leak. |
| `## Per-feature whys` | Required by the reconstruction prompt; not a gold-list leak. |
| `### Accounts` | PLAN domain grouping, not a gold section title. |
| `### Aviary` | PLAN domain grouping, not a gold section title. |
| `### Presence and interactions` | PLAN-derived grouping combining presence and interactions. |
| `### Sync and state` | PLAN-derived grouping. |
| `### Simulation engine` | PLAN-derived grouping. |
| `### Frontend rendering` | PLAN-derived grouping. |
| `### Audio` | PLAN-derived grouping. |
| `### Social visits` | PLAN-derived grouping. |
| `### Accessibility surfaces` | PLAN-derived grouping. |
| `### Security, privacy, performance, and operations` | PLAN-derived grouping. |

The headings do not mirror the gold list's S1-S9 or F1-F40 ordering, nor the source-file grouping in GOLD_WHYS. No heading-mirror concern.

## 1:1 Mapping Suspect

The reconstruction has 10 system-level bullets rather than the gold list's 9 system whys, and its per-feature section follows plan/product domains rather than a neat F1-F40 sequence. It includes many plan features beyond the 40 feature-level gold whys and marks several non-gold features as `NOT RECOVERABLE FROM PLAN`. This is not a 1:1 gold-target map.

## Plan-Derivation Spot Check

1. Reconstruction: "Personality drift is monotonic, additive, and non-negative; absence changes expression through a recency envelope so birds become 'quieter, not mistrustful.'" PLAN support: section 0 names the two-clock model and the same "quieter, not mistrustful" phrase; section 7.4 says the negative-delta path does not exist and recency changes expression only. Assessment: supported.
2. Reconstruction: "The plan uses 'synthetic UUIDs everywhere,' encrypted email columns, no analytics access to simulation rows, aggregate-only telemetry, and synthetic staging calibration." PLAN support: section 0 says "Privacy as architecture" with synthetic UUIDs and firewalled per-account data; sections 12.1-12.2 define the PII firewall and telemetry boundary. Assessment: supported.
3. Reconstruction: "It avoids state-list spam, uses one shared grammar with notebook prose, and prioritizes user-initiated events without flooding the queue." PLAN support: section 11.1 defines naturalist narration, bans state-list formats, shares the grammar with notebook prose, and caps/prioritizes the narration queue. Assessment: supported.

## Verdict: PASS

No significant contamination signs were found. The reconstruction uses plan vocabulary and plan structure, contains no gold IDs, does not map 1:1 to the gold list, and the spot-checked articulate sentences are directly supported by PLAN passages.
