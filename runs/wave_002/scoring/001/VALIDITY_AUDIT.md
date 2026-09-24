# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A mechanical search for gold IDs matching `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` in the frozen reconstruction returned no hits. The reconstruction does not use the gold taxonomy or scorer IDs.

## Vocabulary Check

Sampled phrases and plan derivation:

| Reconstruction phrase | Plan support | Assessment |
| --- | --- | --- |
| "Server is the only writer of personality, mood, perch, and weather" | PLAN invariant I1 uses the same sentence. | Plan-derived |
| "canonical simulation state (server) versus presentation realization (client)" | PLAN section 2.2 states this boundary directly. | Plan-derived |
| "Email is PII and lives in exactly one place" | PLAN invariant I6 uses the same phrase. | Plan-derived |
| "No announcement, gamification, or user-behavior surfaces" | PLAN invariant I7 uses the same phrase. | Plan-derived |
| "designed surface" for reduced motion | PLAN scope says accessibility includes a designed reduced-motion mode; section 8.8 designs the presenter. | Plan-derived |
| "first bird visible in <500 ms" | PLAN scope and performance section include this target. | Plan-derived |
| "load-bearing" | Appears in PLAN decision/risk language and recognizability wording; not unique to the rubric. | Not suspicious |
| "gold", "rubric", "feature-level fidelity", "system-level fidelity" | Mechanical vocabulary search found none of these in reconstruction. | Clean |

## Heading Mirror

The reconstruction has only two headings: `## System-level intent` and `## Per-feature whys`. Those are required by the phase-2A prompt, not mirrored from GOLD_WHYS.md. There are no `###` headings and no headings matching gold-list section titles such as "System-level whys" or "Feature-level whys".

## 1:1 Mapping Suspect

No 1:1 gold-list mapping was observed. The reconstruction has 12 system principles and 80 per-feature items, follows the plan's implementation structure, and does not enumerate S1-S9 or F1-F40. Its order tracks the plan: account/auth, architecture, data model, simulation, sync, visits, frontend, interactions, audio, accessibility, rollout, and tests.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
| --- | --- | --- |
| "Canonical simulation belongs on the server." | PLAN invariant I1 and section 6.1: server-only writer, clients never authoritative. | Supported |
| "Privacy is structural, not a policy afterthought." | PLAN invariant I6, section 12.3 network isolation, aggregate-only telemetry, and PII scanners. | Supported |
| "Accessibility is a first-class aesthetic and parity goal." | PLAN scope: accessibility all at launch; section 8.8 designed reduced motion; section 11.8 pass criterion asks whether the aviary feels alive in each modality. | Supported |

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no gold-list-shaped outline, no scorer/rubric vocabulary, and sampled high-salience claims are directly supported by PLAN.md. The phrase "load-bearing" appeared in the reconstruction, but it also appears in the plan and is used in plan-local engineering prose, so it is not a contamination signal here.
