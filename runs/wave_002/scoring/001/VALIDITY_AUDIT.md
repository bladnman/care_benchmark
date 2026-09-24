# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict: no leakage found. A mechanical search for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` in `RECONSTRUCTION.md` returned no hits.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "engineering invariant" | PLAN §1 and §2 use "engineering invariants with enforcement" | Plan-derived |
| "charm rubric" | PLAN §11.6 and §16.3 use charm rubric language | Plan-derived |
| "load-bearing" | PLAN §1 says "load-bearing decisions" | Plan-derived |
| "Privacy is architectural" | PLAN §1 and §4.7 say the privacy boundary is architectural | Plan-derived |
| "same product" accessibility | PLAN §11 says every user gets the actual product | Plan-derived |
| "NOT RECOVERABLE FROM PLAN" | Required by the phase-2A reconstruction prompt | Acceptable |
| "feature-level fidelity" / "system-level fidelity" / "weight-3" / "multi-layer recovery" | Not present | No concern |

The only grep hit for a scorer-looking word was "charm rubric", which is present in the PLAN.

## Heading Mirror Check

The reconstruction headings mirror the PLAN section groupings: `How to read this plan`, `Executive summary and product guardrails`, `Product invariants and scope`, `Architecture`, `Data model`, `API surface`, `Simulation engine`, `Sync model`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Voice and copy system`, `Accounts, security, and privacy mechanics`, and later rollout/risk/open-question sections. They do not mirror the gold-list headings.

## 1:1 Mapping Suspect Check

No 1:1 mapping to S1-S9 or F1-F40 was observed. The reconstruction has many more per-feature bullets than the 40 gold feature whys, follows PLAN section order, and includes PLAN-only operational details such as deployment topology, data model mechanics, rate limits, and rollout risks.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Assessment |
|---|---|---|
| "The plan makes the simulation a pure, deterministic step function with fixed-point math and a counter-based PRNG." | PLAN §7.1: `step(...)` does no I/O; fixed-point Q16.16; Philox counter-based PRNG | Grounded |
| "Visitors see the same ambient aviary but cannot get offer plans, greeting dispositions, or host presence state." | PLAN §6.4 strips `last_presence_end_at`, `greet`, `offer_plans`, and `offer_cooldown_until` for visitors | Grounded |
| "The data is deliberately not computed so engagement pressure and leaderboards cannot just be exposed." | PLAN §14.4 refuses per-account engagement, visits-per-host, ranking/comparison statistics | Grounded |

## Verdict: PASS

The reconstruction reads as plan-derived. It contains no gold IDs, no free rubric vocabulary, no gold-heading mirroring, and no neat target-list mapping. The sampled articulate claims are grounded in the assigned PLAN.
