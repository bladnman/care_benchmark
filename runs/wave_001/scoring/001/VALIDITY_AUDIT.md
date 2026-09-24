# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: clean. Mechanical grep for `F[0-9]+`, `S[0-9]+`, `R-F[0-9]+`, and `R-S[0-9]+` returned no hits in `RECONSTRUCTION.md`. The same pattern also returned no hits in `PLAN.md`, so there are no unexplained gold-ID echoes.

## Vocabulary Check

Sampled phrases from `RECONSTRUCTION.md` against `PLAN.md`:

| Reconstruction phrase | Plan-derived? | Finding |
|---|---|---|
| "keeping the product's affective promises true at the architecture level" | yes | Exact plan phrase in executive summary. |
| "Canonical state belongs to the server" | yes | Plan defines canonical as server-authored and repeats server-only writer model. |
| "already mid-action" | yes | Plan uses this for first frame and boot renderer. |
| "Presence is measured honestly and counted once" | yes | Exact plan structural choice. |
| "slow, monotonic, and expressive rather than need-based" | yes | Plan repeatedly says slow, monotonic, no need/suffering state. |
| "Everything the user perceives as alive is generated, not stored" | yes | Exact plan structural choice. |
| "a designed register, not motion turned off" | yes | Exact reduced-motion section wording. |
| "no route or credentials into Simulation DB or Identity DB" | yes | Exact architecture/telemetry-plane wording. |
| "slow is fixable, fast is not" | yes | Exact risk/parameter-change rationale. |

No scorer-side terms were found by case-insensitive search for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

## Heading Mirror Check

The reconstruction headings are the required `## System-level intent` and `## Per-feature whys`, followed by plan-shaped `###` headings: How to read this plan, Executive summary and invariants, Scope, Explicit exclusions, Surface inventory and voice register, Architecture, Data model, API surface, Simulation engine, Sync model, Frontend rendering pipeline, Audio pipeline, Accessibility surfaces, Voice and content system, Privacy and security engineering, Performance budgets and observability, Testing strategy, Delivery plan, Rollout, Risks, Decision log, and Open items and parameters.

The first two headings are required by the phase-2A prompt. The remaining headings mirror the candidate PLAN's own sections, not the gold list's system/feature taxonomy. There are no near-exact mirrors of `GOLD_WHYS.md` titles such as "System-level whys," "Feature-level whys," or individual F/S IDs.

## 1:1 Mapping Suspect Check

No 1:1 mapping to the gold targets is visible. The reconstruction does not enumerate S1-S9 or F1-F40, does not use gold IDs, and does not proceed in the gold feature order. It follows the plan sections and contains many plan-only implementation rationales that are not gold rows, which supports plan derivation.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "The plan turns product rules into 'invariants' with 'code, schema, database grant, CI gate, or review gate' enforcement."
   Plan support: section 0 says INV-nn are hard product rules with enforcement mechanisms and that people-being-careful is treated as a gap.

2. Reconstruction sentence: "The plan adds one bounded internal signal, attunement, and fences it in."
   Plan support: section 6.5 says the plan adds `attunement`, bounds it, makes it expression-only, and prevents distress or trait changes.

3. Reconstruction sentence: "Tokens are carried in the URL fragment so they never reach server logs, the edge, or Referer headers."
   Plan support: section 5.6 states visit tokens are in the URL fragment so they never reach server logs, edge, or `Referer` headers.

All three articulate reconstruction sentences have direct plan support.

## Verdict

PASS. The reconstruction reads as plan-derived: no gold IDs, no rubric vocabulary, no gold-heading mirror, no ordered S/F mapping, and sampled rich sentences are grounded directly in the plan. The only minor caveat is that the reconstructor produced a very exhaustive plan-shaped outline, but the outline follows the plan's own section order and vocabulary rather than the held-out scoring taxonomy.
