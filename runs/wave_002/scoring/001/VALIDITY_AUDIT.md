# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found.

Mechanical search of the frozen reconstruction found no gold IDs or external taxonomy tokens matching `F1`-style, `S1`-style, `R-F*`, or `R-S*` forms. The reconstruction uses plan section numbers, ordinary feature names, and plan-derived headings only.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Canonical server truth over client-side ownership" | PLAN says "the server is the only writer" and "The database is authoritative." | Derived from PLAN, not scorer vocabulary. |
| "Account-private, privacy-minimal operation" | PLAN says "account-private initial snapshot," "synthetic UUIDs," and aggregate-only RUM. | Derived summary phrase. |
| "Gentle expressiveness without distress" | PLAN says drift is "one-way toward expressiveness" and neglect never produces distress. | Directly plan-derived. |
| "Naturalist prose instead of game language" | PLAN requires "concise, specific, lowercase naturalist prose" and bans badges/streaks. | Directly plan-derived. |
| "Accessibility is v1 release scope" | PLAN says "Ship accessibility with v1" and calls related work release scope. | Directly plan-derived. |
| "Reliability and performance are launch gates" | PLAN lists release gates for auth, tick latency, rendering, and audio failure. | Derived from rollout section. |
| "Visits are explicit, read-only, and non-present" | PLAN says visitors are read-only and never contribute presence or drift. | Directly plan-derived. |

No rubric-side terms such as `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity` appeared in the reconstruction.

## Heading Mirror

The reconstruction headings are:

- `## System-level intent`
- `## Per-feature whys`
- `### 1. Product boundary and invariants`
- `### 2. Architecture and service boundaries`
- `### 3. Persistent data model`
- `### 4. API and event flow`
- `### 5. Simulation and behavioral engine`
- `### 6. Client rendering pipeline`
- `### 7. Audio pipeline`
- `### 8. Accessibility and performance budgets`
- `### 9. Delivery sequence and rollout`
- `### 10. Risks and mitigations`
- `### 11. Definition of ready to launch`

These mirror the PLAN's own section headings exactly. They do not mirror the gold-list organization (system whys S1-S9, feature whys F1-F40, complete features list). This is expected plan-derived structure, not contamination.

## 1:1 Mapping Suspect

No 1:1 gold mapping pattern found. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve the gold order, and contains far more plan-derived feature bullets than the 49 scored whys. Its per-feature section follows the plan's eleven implementation sections rather than the gold catalog's files and IDs.

## Plan-Derivation Spot Check

1. Reconstruction: "clients present and request, while state, sequencing, drift, and mood are advanced in one serialized place."
   PLAN support: "the server is the only writer of personality and mood; clients render snapshots and submit events," plus "The scheduled tick remains the only canonical simulation writer."
   Assessment: supported.

2. Reconstruction: "Telemetry, logs, analytics, visits, and presence are all shaped to avoid exposing account behavior."
   PLAN support: synthetic UUIDs, encrypted email, aggregate-only RUM, no raw pointer/key data, and visitor activity excluded from simulation/analytics identity.
   Assessment: supported.

3. Reconstruction: "Reduced-motion is a designed alternate renderer."
   PLAN support: "Reduced-motion is a designed alternate renderer" with slow cross-fades, no leaf drift, and continued bird drift/mood/calls.
   Assessment: directly supported.

## Verdict: PASS

The reconstruction reads as plan-derived. It mirrors the PLAN's section structure, contains no gold IDs or rubric vocabulary, and its strongest summary phrases can be grounded in the PLAN. The audit found no significant contamination signatures.
