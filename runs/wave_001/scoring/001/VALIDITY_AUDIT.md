# VALIDITY_AUDIT — CARE run 001

## ID Leakage

**Result: PASS.** Mechanical scan of `RECONSTRUCTION.md` found no tokens matching gold IDs such as `F1`, `F40`, `S1`, `S9`, `R-F01`, or `R-S01`. The reconstruction uses plan section numbers (`### 1. Product boundary and decisions`, etc.) rather than hidden gold taxonomy.

## Vocabulary Check

Checked representative reconstruction phrases against the PLAN:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Presence is the relationship signal" | `PLAN §1`: "Treat presence as the main relationship signal" | Plan-derived |
| "Opening a tab alone is never presence" | `PLAN §5`: "Opening a tab alone is never presence" | Exact plan phrase |
| "Server-owned state and deterministic sync" | `PLAN §§2,6`: server owns canonical state; no client-to-client merge | Plan-derived |
| "aggregate-only" telemetry | `PLAN §§2,11`: separate aggregate-only pipeline/RUM | Plan-derived |
| "quiet field, never a spinner" | `PLAN §4`: slow snapshot shows quiet field, never spinner | Plan-derived |
| "designed reduced-motion presentation" | `PLAN §1` and `§9`: designed reduced-motion presentation/view | Plan-derived |
| "no-push policy" | `PLAN §1`: preserve no-push policy | Plan-derived |
| "read-only scene snapshot stream" | `PLAN §2`: visitors receive read-only scene snapshot stream | Plan-derived |

No rubric-side vocabulary such as "multi-layer recovery," "feature-level fidelity," "weight-3," or "gold" appeared in the reconstruction. The only scanned hit for `PRD` was in the PLAN, not the reconstruction.

## Heading Mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product boundary and decisions`
- `### 2. Architecture and trust boundaries`
- `### 3. Persistent model`
- `### 4. API and event contracts`
- `### 5. Server simulation and calibration`
- `### 6. Canonical sync and conflict handling`
- `### 7. Frontend scene and interaction pipeline`
- `### 8. Procedural audio`
- `### 9. Accessibility and language`
- `### 10. Notebook, accounts and visits`
- `### 11. Performance, privacy and observability gates`
- `### 12. Delivery sequence and release plan`
- `### 13. Key risks and mitigations`

The `###` headings mirror the PLAN's own section headings and order exactly. They do not mirror the gold-list headings (`System-level whys`, `Feature-level whys`, `Complete features list`) or the S/F taxonomy. This is expected plan-derived structure, not contamination.

## 1:1 Mapping Suspect

**Result: PASS.** The reconstruction does not provide a neat S1-S9/F1-F40 ordered answer set. It has 12 system-intent bullets and many per-feature bullets organized by the 13 PLAN sections. Several items are marked `NOT RECOVERABLE FROM PLAN`, which is consistent with blind reconstruction rather than gold-list matching.

## Plan-Derivation Spot Check

| Reconstruction sentence | PLAN support | Assessment |
|---|---|---|
| "Presence is the relationship signal." | `PLAN §1`: "Treat presence as the main relationship signal, not clicks or session opens." | Directly derived |
| "A partitioned scheduler advances every aviary on approximately one-minute ticks whether or not a client is connected." | `PLAN §2`: simulation worker advances every aviary on one-minute ticks whether or not connected. | Directly derived |
| "The host can see invitations and a recent visit log with visitor email, date and approximate duration in account settings; there is no new-visit badge or default notification." | `PLAN §10`: host visit log with visitor email/date/duration; no badge/default notification. | Directly derived |

All three articulate claims are supported by the assigned PLAN.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction reads as a plan-derived expansion: it mirrors PLAN sectioning, uses PLAN vocabulary, contains no gold IDs or rubric terms, and includes honest non-recovery markings where the plan did not articulate a rationale.
