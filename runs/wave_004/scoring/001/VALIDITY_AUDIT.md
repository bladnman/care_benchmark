# VALIDITY_AUDIT — CARE run 001

## ID Leakage

**Result: PASS.** A targeted search found no gold IDs or rubric-style IDs in the frozen reconstruction: no `S1`-`S9`, `F1`-`F40`, `R-Fxx`, `weight-2`, `weight-3`, `intent fidelity`, or `feature-level fidelity` strings. The reconstruction uses ordinary plan-derived feature names and several `NOT RECOVERABLE FROM PLAN` markers, but no gold-list identifiers.

## Vocabulary Check

Sampled load-bearing phrases from `RECONSTRUCTION.md` against `PLAN.md`:

| Reconstruction phrase | PLAN support | Result |
|---|---|---|
| "designed surface, not a fallback" | `Reduced-motion rendering as a designed surface, not a fallback` | plan-derived |
| "empirical limit for call signature recognizability" | `Cap at seven birds (empirical limit for call signature recognizability)` | plan-derived |
| "monotonic toward expressive" | `Monotonic toward expressive` | plan-derived |
| "canned audio concerns" | `bundle budget and canned audio concerns` | plan-derived |
| "matter-of-fact" | multiple named exceptions in PLAN | plan-derived |
| "visibility + focus + recent activity" | presence detection and risk mitigation sections | plan-derived |
| "server is sole writer" | Sync Model section | plan-derived |
| "engagement metrics" | Rollout Constraints section | plan-derived |
| "no co-presence affordances" | Visit Feature Misuse mitigation | plan-derived |
| "quiet field" | Loading Sequence section | plan-derived |

No suspicious gold-side vocabulary appeared. The phrase `NOT RECOVERABLE FROM PLAN` is an expected phase-2A reconstruction marker, not evidence of gold leakage.

## Heading Mirror

Reconstruction headings:

| Reconstruction heading | Closest PLAN/GOLD relationship | Finding |
|---|---|---|
| `## System-level intent` | Required reconstruction structure; resembles scorer concepts but was part of phase-2A format | acceptable |
| `## Per-feature whys` | Required reconstruction structure | acceptable |
| `### V1 scope and explicit boundaries` | Mirrors PLAN `Scope` and non-goals organization | plan-derived |
| `### Architecture, data, and API surface` | Mirrors PLAN architecture/data/API sections | plan-derived |
| `### Simulation engine and sync model` | Mirrors PLAN sections 5 and 6 | plan-derived |
| `### Frontend rendering pipeline` | Mirrors PLAN section 7 | plan-derived |
| `### Audio pipeline` | Mirrors PLAN section 8 | plan-derived |
| `### Accessibility surfaces` | Mirrors PLAN section 9 | plan-derived |
| `### Performance, observability, rollout, and risks` | Mirrors PLAN sections 10-12 | plan-derived |

The headings mirror the PLAN structure, not `GOLD_WHYS.md` section titles. No heading-level gold-list echo was found.

## 1:1 Mapping Suspect

**Result: PASS.** The reconstruction does not produce a neat S1-S9 or F1-F40 sequence. It has 8 system bullets, not 9; its per-feature items follow the candidate PLAN's implementation sections and include many non-gold features as well as several `NOT RECOVERABLE FROM PLAN` entries. That shape is consistent with a plan-derived reconstruction rather than a gold-list-aligned reconstruction.

## Plan-Derivation Spot Check

| Reconstruction sentence | Supporting PLAN passage | Finding |
|---|---|---|
| "The aviary is designed to continue, not reset, when the user is away." | Loading Sequence: `Server-side simulation tick ensures aviary continues without viewer`; Mood risk: `aviary feels reset, not continued` | grounded |
| "Canonical state and sync correctness belong on the server." | Sync Model: `Server is sole writer`; `All clients read from same canonical state`; `No client-side state to merge` | grounded |
| "Accessibility is a designed surface, not a fallback." | Render Pipeline Boundary and Accessibility Risks use `designed surface, not a fallback` and `first-class design surface, not checklist` | grounded |

The most articulate reconstruction claims are directly traceable to PLAN wording.

## Verdict

**PASS.** I found no significant contamination signatures. The reconstruction reads as derived from the PLAN: its vocabulary and headings closely follow PLAN sections, it lacks gold IDs and rubric language, and it does not align 1:1 with the gold why list. Scores should be treated as valid for this run.
