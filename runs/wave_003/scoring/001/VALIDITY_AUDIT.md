# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it mirrors PLAN section order, quotes PLAN-specific implementation phrases, contains no gold ID leakage, and does not map neatly to S1-S9/F1-F40.

## ID Leakage

No gold-ID leakage found. A targeted search of `RECONSTRUCTION.md` found no `F1`-`F40`, `S1`-`S9`, `R-Fxx`, `canonical #`, weight-tier, multi-layer, gold-list, or intent-fidelity terminology. The PLAN likewise does not contain those IDs, so there are no offending ID hits to cross-reference.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "persistent, browser-based ambient aviary" | PLAN section 1, opening sentence | Plan-derived exact phrase |
| "slow, observational relationships" | PLAN section 1, opening sentence | Plan-derived exact phrase |
| "HARD BOUNDARY" | PLAN architecture telemetry block | Plan-derived exact phrase |
| "robotic ARIA attribute updates" | PLAN section 9.2 | Plan-derived exact phrase |
| "First Frame Mid-Action" | PLAN section 7.2 heading | Plan-derived exact phrase |
| "Zero External Media Requests" | PLAN section 10.2 heading | Plan-derived exact phrase |
| "Last-Write-Wins" | PLAN sections 1.1 and 6.1 | Plan-derived exact phrase |
| "no false warmth" | PLAN section 9.1 Matter-of-Fact voice rules | Plan-derived exact phrase |
| "engine-equivalent to tab-close" | PLAN section 1.1 Settle bullet | Plan-derived exact phrase |
| "background aliveness" | PLAN section 8.3 listen-in decay | Plan-derived exact phrase |

The phrase `NOT RECOVERABLE FROM PLAN` is instruction-scaffold vocabulary rather than PLAN prose, but it is expected in phase-2A reconstruction output and is not gold/rubric leakage by itself.

## Heading Mirror

`RECONSTRUCTION.md` has two markdown headings: `## System-level intent` and `## Per-feature whys`. These are the expected reconstruction structure, not gold-list section titles. Its bold subsection headings mirror PLAN section order: `Executive Summary & Scope Boundary`, `Explicit Non-Goals`, `System Architecture & Service Topology`, `Data Model & Database Schemas`, `API Surface & Protocols`, `Simulation Engine Design`, `Multi-Device Synchronization & Conflict Resolution`, `Frontend Rendering Pipeline`, `Procedural Audio Pipeline`, `Accessibility Surfaces & UX Voice Architecture`, `Performance Budgets`, `Rollout`, `Technical Risk Matrix`, and `Verification Checklist`. They do not mirror the gold taxonomy order or titles.

## 1:1 Mapping Suspect

No 1:1 gold mapping pattern found. The reconstruction gives eight system-level intent bullets rather than exactly nine S-level items, and the per-feature list follows PLAN section order with many more than 40 entries. Several gold-bearing features are merged into broader PLAN-derived bullets, while many non-gold PLAN implementation details receive entries. That shape is consistent with derivation from PLAN, not from `GOLD_WHYS.md`.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "The architecture treats the client as presentation and input capture only." | PLAN section 6.1: clients are strictly render and input capture nodes and do not submit state. | Supported |
| "Observability has a 'HARD BOUNDARY' and 'Zero per-bird, per-vector, or per-user interaction events' in analytics." | PLAN telemetry diagram and section 10.3 prohibit per-bird/per-user analytics. | Supported |
| "Rather than emitting robotic ARIA attribute updates, the system maintains an aria-live live region populated with running naturalist prose." | PLAN section 9.2 contains the same ARIA/naturalist narration design. | Supported |

## Verdict: PASS

No significant contamination signatures appear. The reconstruction sometimes over-compresses rationale and sometimes honestly marks items not recoverable, but those are scoring issues, not contamination. It appears derived from PLAN and the phase-2A reconstruction scaffold rather than from the gold list or rubric.
