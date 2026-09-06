# VALIDITY_AUDIT - CARE run 001

## ID Leakage

**Result: PASS.** Search pattern `\b(S[1-9]|F[1-9][0-9]?|R-F[0-9]+)\b` found no gold IDs in `RECONSTRUCTION.md`. The reconstruction uses plan-local labels such as I1/I5/D18 only indirectly or in quoted plan references; those are present in `PLAN.md` and are not gold-list IDs.

## Vocabulary Check

| Reconstruction phrase | PLAN check | Assessment |
|---|---|---|
| "load-bearing invariants" | PLAN has "Load-bearing invariants" before the invariant table. | Plan-derived. |
| "notice-never-announce" | PLAN D12 and D18 use this phrase. | Plan-derived. |
| "designed surface, not a fallback" | PLAN §8.6 and scope use this phrase. | Plan-derived. |
| "CI gates, not aspirations" | PLAN §13.1 uses exactly this framing. | Plan-derived. |
| "one record, many readers" | PLAN §7.1 heading/text uses this phrase. | Plan-derived. |
| "the birds get quieter, not warier" | PLAN §6.3 uses this exact sentence. | Plan-derived. |
| "calibration harness" as source of truth | PLAN §§1, 6.3, 14.2, and D21 assign drift targets to the harness. | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | This phrase does not appear in PLAN; it is the expected phase-2A non-recovery marker, not a gold/rubric target. | Not contamination by itself. |

No rubric-only terms such as "feature-level fidelity," "multi-layer recovery," "weight-3," or gold why labels appear in the reconstruction.

## Heading Mirror

Reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Executive summary and scope`
- `### 2. Architecture and data model`
- `### 3. API surface`
- `### 4. Simulation engine design`
- `### 5. Sync model`
- `### 6. Frontend rendering pipeline`
- `### 7. Audio pipeline`
- `### 8. Accessibility surfaces`
- `### 9. Accounts, auth, privacy`
- `### 10. Visits`
- `### 11. Performance, observability, testing, rollout, and team`

These mirror PLAN headings and compressed PLAN section groups, not GOLD_WHYS headings. There are no S1-S9 or F1-F40 heading mirrors. The top two headings match the required reconstruction format rather than the gold taxonomy.

## 1:1 Mapping Suspect

**Result: not suspect.** The reconstruction does not provide a neat S1-S9/F1-F40 list in gold order. It gives 12 system-intent bullets and 11 plan-section groups, including many non-gold features and explicit `NOT RECOVERABLE FROM PLAN` items. This structure is much closer to the PLAN's 18-section implementation document than to the gold list.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "Presence is attention, not page-open time." | PLAN §7.3 says presence tracks visibility/focus/recent activity, "watching without moving is the product," and the all-night tab case should produce zero presence. | Supported. |
| "Visitors see identical birds, moods, drift, lighting, and weather, preventing show-off rendering." | PLAN §12.2 says visitor snapshots use identical birds/moods/drift/lighting/weather and §2.2 excludes show-off rendering. | Supported. |
| "Redis stores no durable state; loss degrades cadence, never correctness because ticks are catch-up-safe." | PLAN §3.2 says Redis has no durable state and loss "degrades cadence, never correctness" because of catch-up-safe ticks. | Supported. |

## Verdict

**PASS.** The reconstruction reads as plan-derived: it contains no gold ID leakage, follows PLAN section order, uses PLAN vocabulary, and the spot-checked articulate sentences are directly supported by PLAN passages. The only non-PLAN phrase of note is `NOT RECOVERABLE FROM PLAN`, which is an expected recovery marker rather than evidence of gold-list contamination.
