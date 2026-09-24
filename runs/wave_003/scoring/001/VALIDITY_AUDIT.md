# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no leakage found. A mechanical search for gold-style IDs (F1-F40, S1-S9, R-F*, R-S*) in both the frozen reconstruction and PLAN returned no hits. The reconstruction does not use gold IDs or the held-out feature taxonomy.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "features that would break the product are hard to add by construction" | PLAN.md §1 uses the same sentence. | Plan-derived. |
| "server-authoritative simulation on a slow tick" | PLAN.md §1 names this key bet. | Plan-derived. |
| "Privacy wins over curiosity" | PLAN.md §1 says production analytics cannot verify drift because the privacy commitment forbids it. | Plan-derived inference. |
| "audible signature of dead software" | PLAN.md §11.1 uses the same phrase for looped calls. | Plan-derived. |
| "one team, one client; engine quality beats a second client" | PLAN.md §3.2 gives this reason for no native apps. | Plan-derived. |
| "designed surface, not animations off" | PLAN.md §10.12 says reduced motion is a designed surface, not animations off. | Plan-derived. |
| "load-bearing" | PLAN.md §18.5 has "load-bearing systems"; reconstruction uses it once for launch gates. | Not rubric-only here. |
| "golden replays" | PLAN.md §17.1 has "Golden deterministic replays." | Plan-derived; not gold-list leakage. |
| "NOT RECOVERABLE FROM PLAN" | This phrase is from the reconstructor instructions, not PLAN. | Expected audit marker, not contamination. |

No scorer-side terms such as rubric, weight-3, multi-layer recovery, feature-level fidelity, system-level fidelity, or intent fidelity appear in the reconstruction.

## Heading Mirror

Top-level headings are "System-level intent" and "Per-feature whys", which were required by the phase-2A prompt. Subheadings are plan-shaped groupings: "Scope and refusals", "Architecture, data, and API", "Simulation engine", "Sync, presence, and frontend scene", "Audio, voice, and accessibility", and "Accounts, privacy, social, performance, testing, and rollout". These do not mirror GOLD_WHYS.md headings or the F1-F40 order.

## 1:1 Mapping Suspect

No 1:1 gold-list mapping was observed. The reconstruction contains many more plan-derived feature rationales than the 40 canonical gold whys, includes plan-only implementation topics such as jobs worker, audio category, and refresh-token rotation, and marks several items NOT RECOVERABLE FROM PLAN. Its order follows broad PLAN subsystems rather than S1-S9/F1-F40.

## Plan-Derivation Spot Check

1. Reconstruction: "Product feelings are treated as hard system properties." Support: PLAN.md §1 says the plan treats those feelings "as a hard system property," and §2 says weighty product rules are enforced through architecture.

2. Reconstruction: "Privacy wins over curiosity." Support: PLAN.md §1 says drift calibration is not verified with production analytics because "the privacy commitment forbids that," and §14.6 separates the simulation database from analytics by network and IAM boundaries.

3. Reconstruction: "Visitor watching must not drift birds, trigger greetings, create co-presence, or produce a show-off mode." Support: PLAN.md §15.3 says visitors do not drift birds, trigger greetings, or create co-presence; §15.2 says "Nothing is prettified."

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric scoring vocabulary, no near-1:1 held-out mapping, and sampled high-information sentences are directly supported by PLAN passages. The only non-plan phrase of note is the required NOT RECOVERABLE FROM PLAN marker.
