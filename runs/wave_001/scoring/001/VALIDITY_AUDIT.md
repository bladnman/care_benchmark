# VALIDITY_AUDIT - CARE run 001

## Verdict

PASS. The frozen reconstruction reads as plan-derived: it contains no gold ID leakage, no scorer/rubric vocabulary, and its headings follow the candidate plan's own structure rather than the gold list. Several entries are honestly marked `NOT RECOVERABLE FROM PLAN`, which is a validity-positive signal rather than a contamination sign.

## ID Leakage

No gold IDs or external gold taxonomy tokens were found in `RECONSTRUCTION.md` using the leakage pattern `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, or `R-S[0-9]+`. Because there were no hits, there was nothing to cross-check against `PLAN.md`.

## Vocabulary Check

No scorer-side vocabulary hits were found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `intent fidelity`, or `load-bearing`.

Plan-derived phrase spot checks:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Continuity over application shell" | PLAN outcome says the product must preserve "continuity" and that birds, not the shell, notice the user. | Plan-derived synthesis. |
| "Server-owned canonical life" | PLAN invariant says the server owns one canonical aviary and clients submit facts. | Plan-derived synthesis. |
| "Relationship privacy and non-exposure" | PLAN has render-safe snapshots, hidden vectors, aggregate-only telemetry, and no joins to simulation events. | Plan-derived synthesis. |
| "Presence means qualifying attention" | PLAN requires visibility, focus, and recent pointer/key activity. | Plan-derived synthesis. |
| "Accessibility as affective parity" | PLAN says accessible journeys must retain the aviary's affective core and ships designed reduced motion. | Plan-derived synthesis. |
| "Bounded architecture with fewer failure modes" | PLAN explicitly says the chosen PostgreSQL/outbox/row-lock shape provides correctness with fewer failure modes. | Directly plan-derived. |

## Heading Mirror

The only `##` headings are `System-level intent` and `Per-feature whys`, which are required by the phase-2A reconstruction prompt. The `###` headings mirror the candidate plan's sections: `Outcome and product invariants`, `Architecture and ownership boundaries`, `Data model`, `API surface and contracts`, `Authentication, authorization, and privacy controls`, `Simulation engine`, `Multi-device consistency and offline behavior`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility implementation`, `Performance budgets and enforcement`, and `Delivery sequence, rollout, and operations`.

These headings do not mirror `GOLD_WHYS.md` sections such as `System-level whys`, `Feature-level whys`, or the S/F identifier taxonomy. They are consistent with a plan-derived reconstruction.

## 1:1 Mapping Suspect

No 1:1 mapping to S1-S9 or F1-F40 is present. The reconstruction has 12 system-intent bullets and many plan-structured per-feature bullets, not a neat sequence of 9 system whys and 40 feature whys. It does not use gold IDs or the gold ordering. This is not suspect.

## Plan-Derivation Spot Check

1. Reconstruction: "Clients submit facts about interactions, never desired trait values."
   PLAN support: section 1 states that clients submit facts about interactions and never desired trait values.
   Assessment: directly supported.

2. Reconstruction: "Visits are explicit, revocable, read-only, and do not contribute presence or interaction input."
   PLAN support: section 1 has the same invariant, and sections 5-6 define separate visit grants/snapshots with no host mutation or presence routes.
   Assessment: directly supported.

3. Reconstruction: "Append-only interaction rows plus an outbox and row-level tick claims provide the required correctness with fewer failure modes."
   PLAN support: section 3 explicitly rejects Kafka/event-sourcing/client DB and gives this exact architectural rationale.
   Assessment: directly supported.

## Verdict: PASS

The reconstruction shows no meaningful contamination signatures. It is detailed, but its detail is explained by the unusually detailed plan; the structure, vocabulary, and omissions all point to plan derivation rather than exposure to scorer-side gold material.
