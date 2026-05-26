# VALIDITY_AUDIT - CARE run 001

## 1. Gold ID leakage check

Verdict for this check: no leakage found.

I searched the frozen reconstruction and PLAN for gold-side identifiers and rubric terms such as `S1`, `F1`, `R-F01`, `weight-3`, `multi-layer`, `feature-level`, `intent fidelity`, `gold`, and `rubric`. There were no hits in either file. The reconstruction does not use gold IDs, PRD file labels, or the S/F taxonomy.

## 2. Vocabulary check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "v1 boundary and non-goals respect" | PLAN heading: "Scope - v1 Definition and Non-Goals Respect" | Plan-derived. |
| "Recognizability over scale" | PLAN: "Bird count caps at 7 because call signatures must remain individually recognizable" | Synthesis from PLAN, not gold leakage. |
| "Canonical server state over client mutation" | PLAN: "single writer," "Client never mutates canonical state," "server tick is sole writer" | Plan-derived. |
| "Expressive drift without neglect punishment" | PLAN: "monotonic toward expressive," "no negative drift on neglect" | Plan-derived. |
| "Strict presence as a product and calibration boundary" | PLAN: strict visible + focused + recent pointer/key; presence-signal inflation risk | Plan-derived synthesis. |
| "designed surfaces, not parity checklists" | PLAN Risks: accessibility surfaces, "not parity checklists" | Exact PLAN vocabulary. |
| "aggregate-only observability" | PLAN Performance/Rollout: aggregate-only RUM and metrics | Plan-derived. |
| "NOT RECOVERABLE FROM PLAN" | Phase-2A task vocabulary, not gold taxonomy | Not contamination by itself. |

No sampled phrase reads like hidden access to `GOLD_WHYS.md`. The phrasing is mostly compressed from PLAN headings and bullets.

## 3. Heading mirror check

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`

These headings match the expected reconstruction shape, not the gold-list section titles. They do not mirror specific S1-S9 or F1-F40 titles. The bullets under `Per-feature whys` follow the PLAN's section order (Scope, Architecture, Simulation, Sync, Frontend, Audio, Accessibility, Performance, Rollout), not the gold list's exact S/F order.

## 4. 1:1 mapping suspect check

No 1:1 gold mapping pattern found. The reconstruction does not enumerate S1-S9 or F1-F40, does not use the gold IDs, and includes many plan-derived non-gold items such as magic-link auth, snapshot service, browser support, p99 tick latency, WCAG chrome, and no per-call allocation leaks. Its order mirrors the PLAN structure rather than the gold list.

## 5. Plan-derivation spot check

1. Reconstruction: "The plan repeats that the server tick is the 'single writer' and 'sole writer of personality/mood,' clients 'write only events,' and 'Client never mutates canonical state.'"
   - PLAN support: Architecture says "single writer of personality state" and "Client never mutates canonical state"; Sync says "Server tick is sole writer" and "Clients write only events." Supported.

2. Reconstruction: "The risk section says lax presence would cause 'presence-signal inflation' and 'leaks drift speed across population.'"
   - PLAN support: Risks and mitigations includes "Presence-signal inflation" and says a lax definition "leaks drift speed across population." Supported.

3. Reconstruction: "The plan says procedural audio enables 'real-time variation on every call,' true chorus overlap, and avoids recorded loops that 'fail chorus.'"
   - PLAN support: Audio Pipeline says "Real-time variation on every call" and chorus mixing; Risks says recorded loops fail chorus. Supported.

## 6. Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no neat S/F enumeration, and its headings/items follow the PLAN rather than the gold list. The main issue is not contamination; it is conservative or lossy reconstruction, with many feature whys marked not recoverable or reduced to rule-only summaries.
