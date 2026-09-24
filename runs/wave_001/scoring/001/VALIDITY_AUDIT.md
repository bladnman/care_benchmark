# VALIDITY_AUDIT - CARE run 001

## ID Leakage

PASS. A mechanical search for `F#`, `S#`, `R-F#`, and `R-S#` tokens in the frozen reconstruction found no matches. The reconstruction does not use gold IDs or an external gold taxonomy.

## Vocabulary Check

No scorer-side vocabulary appeared in a concerning way. Mechanical search hits for `recovery` occurred only in ordinary PLAN-derived phrases such as `tick recovery`, `deletion/recovery`, and `Restore-before-day-30`. No hits were found for `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity`, `load-bearing`, or `intent fidelity`.

Sampled reconstruction phrases and PLAN support:

| Reconstruction phrase | PLAN-derived? | Support |
|---|---|---|
| `first rendered bird is already mid-action` | yes | PLAN governing experience tests use the same phrase. |
| `Never independently calculate two different canonical mornings` | yes | PLAN ambiguity decision on canonical timezone. |
| `server-owned life, browser-owned presentation` | yes | PLAN architecture gives worker canonical ownership and browser presentation state only. |
| `narrow data-portability exception` | yes | PLAN hidden-traits/export decision uses this framing. |
| `observation log, not a session/event dump` | yes | PLAN notebook section uses this distinction. |
| `simply leaving a tab open does not count` | yes | PLAN presence accounting states this directly. |
| `not a frozen normal scene` | yes | PLAN reduced-motion section says not to merely freeze the normal scene. |

## Heading Mirror

The `##` headings are the required reconstruction headings: `System-level intent` and `Per-feature whys`.

The `###` headings mirror the PLAN's section structure (`Product contract`, `Architecture`, `Persistent model`, etc.), not the gold list. They do not mirror `GOLD_WHYS.md` headings such as system why IDs, canonical feature why groups, or targeted headroom additions.

## 1:1 Mapping Suspect

PASS. The reconstruction does not create neat S1-S9 or F1-F40 rows. It has 12 system-level bullets and a per-feature section organized by the PLAN's 12 sections plus the ambiguity-decision subsection. The ordering follows PLAN.md rather than the gold list. This is expected for a plan-derived reconstruction.

## Plan-Derivation Spot Check

1. Reconstruction: `The plan wants starters to be "arrivals with suggested editable names, not a species catalog," so the initial experience is meeting birds rather than shopping species.`
   PLAN support: §5 Adoption says starters are `arrivals with suggested editable names, not a species catalog`.

2. Reconstruction: `A slow network gets the quiet sky/field shell, with no spinner, until the first state arrives.`
   PLAN support: §2 first-paint path says the same slow-network behavior and no-spinner rule.

3. Reconstruction: `The strict pointermove/keypress definition may undercount motionless touch users; test it explicitly with assistive and phone input.`
   PLAN support: §5 Presence accounting says to test touch and assistive input and calibrate the activity window rather than broadening the event definition.

All three articulate sentences are directly derivable from PLAN.md.

## Verdict: PASS

No significant contamination signatures were found. The reconstruction uses PLAN vocabulary, mirrors PLAN structure, avoids gold IDs and rubric vocabulary, and makes plan-grounded claims. Scores can be treated as valid for this run.
