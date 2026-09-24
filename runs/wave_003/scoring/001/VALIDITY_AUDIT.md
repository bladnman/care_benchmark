# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Mechanical search for `F#`, `S#`, `R-F#`, and `R-S#` found one apparent hit: `S3` in the phrase **"S3 exports with SSE-KMS and lifecycle deletion"**. This is an AWS S3 storage reference, not gold ID leakage. The same storage reference appears in PLAN (`PLAN.md` lines around the architecture/storage/export sections). No gold IDs, rebuild IDs, or external gold taxonomy were found in the frozen reconstruction.

Verdict for this check: PASS.

## Vocabulary Check

Sampled reconstruction phrases and plan derivation:

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Invariants are the contract" | PLAN §0/§1 says invariants are the contract and violations are out of scope. | Plan-derived |
| "server simulation is canonical; clients are views" | PLAN §4.2 and §8.1 use the same client/server split. | Plan-derived |
| "Growth is monotonic, slow, and non-punitive" | PLAN INV-2, §7.4, §7.6, and risks R-1/R-2. | Plan-derived |
| "announce, count, rank, or reach for the user" | PLAN INV-9 and §2.2 refuse toasts, counters, ranking, push, and re-engagement. | Plan-derived synthesis |
| "Voice is split by surface" | PLAN §0 and INV-12 define product vs. system surfaces. | Plan-derived |
| "Privacy is structural and aggregate-only" | PLAN INV-10/INV-11, §13.7, and §14.6. | Plan-derived |
| "first contact should feel alive, not loaded" | PLAN INV-13 and §9.1 first-frame/quiet-field/no-spinner boot path. | Plan-derived synthesis |
| "load-bearing constraints" | PLAN has "Load-bearing invariants" as a heading. | Plan-derived |

Rubric-side vocabulary search found no `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, or `system-level fidelity`. The only searched term hit was `load-bearing`, which is native to PLAN, not contamination.

## Heading Mirror

Reconstruction headings:

- `## System-level intent`
- `## Per-feature whys`
- `### Scope, refusals, and decisions`
- `### Architecture, data, and API`
- `### Simulation engine`
- `### Sync, frontend, audio, and voice`
- `### Accessibility, privacy, observability, and rollout`

The first two headings are required by the phase-2A reconstruction prompt. The `###` headings mirror the PLAN's broad organization, not the held-out gold list. They do not echo `GOLD_WHYS.md` section titles such as `System-level whys`, `Feature-level whys`, or the F/S target taxonomy.

## 1:1 Mapping Suspect

No 1:1 mapping to the gold targets was found. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve the gold order, and includes many plan-native items beyond the 49 scored whys. Its structure follows the PLAN's sections and decision/invariant vocabulary.

## Plan-Derivation Spot Check

1. Reconstruction: "The product should not announce, count, rank, or reach for the user."
   PLAN support: INV-9 bans toasts, badges, counters, spinners, push, and aviary emails; §2.2 refuses gamification, notifications, public discovery, leaderboards, and social-network surfaces.

2. Reconstruction: "Presence means active attention, credited conservatively."
   PLAN support: INV-5 defines presence as visibility + focus + recent pointer/key activity; D-7 names over-counting as the worse failure; §7.3 unions overlapping device intervals.

3. Reconstruction: "Privacy is structural and aggregate-only."
   PLAN support: INV-10 stores email once and encrypted; INV-11 says telemetry has no account, bird, session, or email dimension; §13.7 blocks telemetry/ETL routes into `sim`; §14.6 lists deliberately unmeasured private behavior.

All three articulate plan-derived ideas with no need to consult held-out gold material.

## Verdict: PASS

The frozen reconstruction reads as plan-derived. There is one mechanical false-positive ID hit (`S3` as AWS S3) and one plan-native phrase (`load-bearing`), but no gold taxonomy leakage, scorer vocabulary, heading mirror, or neat gold-order mapping. Scores can be treated as valid for this run.
