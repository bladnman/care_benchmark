# VALIDITY_AUDIT - CARE run 001

## Gold ID Leakage Check

Verdict for this check: no leakage found. The frozen reconstruction does not use gold identifiers such as `F1`, `F40`, `S1`, `S9`, or `R-F01`. I also checked the assigned PLAN for the same ID patterns; no matching gold-ID vocabulary appears there either. The reconstruction uses product vocabulary and plan section language rather than rubric row identifiers.

## Vocabulary Check

| Reconstruction phrase | Appears in PLAN? | Assessment |
|---|---|---|
| "load-bearing" | yes - PLAN opening calls several surfaces "load-bearing" | Not suspect |
| "already in motion conceit" | yes - PLAN performance/risk language | Not suspect |
| "notice not announce" | yes - PLAN intro and risk sections | Not suspect |
| "aggregate-only" | yes - PLAN privacy/observability sections | Not suspect |
| "matter-of-fact" | yes - PLAN voice/API/error sections | Not suspect |
| "first-class, not checklist" | yes - PLAN accessibility scope | Not suspect |
| "not animations off" | yes - PLAN reduced-motion scope | Not suspect |
| "no LWW" | yes - PLAN scope/sync model | Not suspect |
| "social surface pressure" | yes - PLAN visit risk section | Not suspect |
| "rule/invariants" language | yes - PLAN calls refusals hard invariants | Not suspect |

The only rubric-adjacent phrase is "load-bearing", but it is prominent in PLAN itself and is not a contamination signal here.

## Heading Mirror Check

The reconstruction headings are `## System-level intent`, `## Per-feature whys`, then plan-shaped subsections such as `Scope - v1 inclusions and explicit exclusions`, `Architecture`, `Data model`, `API surface`, `Simulation engine design`, `Sync model and conflict handling`, `Frontend rendering pipeline`, `Audio pipeline`, `Accessibility surfaces`, `Performance budgets, observability, and CI gates`, `Rollout strategy`, `Testing strategy`, `Security and privacy engineering`, `Risks, mitigations, and calibration`, and `Open questions and calls made`.

These headings mirror the PLAN's section structure, not `GOLD_WHYS.md` section headings. There is no heading-by-heading echo of S1-S9 or F1-F40.

## 1:1 Mapping Suspect Check

No near-1:1 gold mapping is present. The reconstruction does not enumerate S1-S9 or F1-F40, does not preserve gold order, and includes many plan features without feature-level gold whys. Its per-feature list follows the PLAN's product sections and engineering sections. That structure is expected from plan derivation.

## Plan-Derivation Spot Check

1. Reconstruction sentence: "Privacy boundaries are architectural, not policy hopes."
   PLAN support: Section 10.3 says telemetry pipelines are "physically and logically separated" from the simulation DB and that there is "No bridge" leaking vectors, names, or event sequences. Section 13 repeats aggregate-only-by-construction privacy.

2. Reconstruction sentence: "The product should prefer noticing over announcing."
   PLAN support: Section 1.2 excludes welcome toasts, modals, banners, notifications, counters, and visit-frequency surfaces. Section 1.1 makes return greeting procedural and staggered; Section 7.4 makes top-bar chrome fade.

3. Reconstruction sentence: "Accessibility is a designed mode, not a fallback."
   PLAN support: Section 1.1 says accessibility is "first-class, not checklist" and reduced motion is "not 'animations off'". Section 7.3 defines a separate cross-fade renderer, and Section 9 covers narration, captions, keyboard, contrast, and screen-reader behavior.

All three articulate sentences have direct PLAN support.

## Verdict: PASS

The reconstruction reads as plan-derived. I found no gold ID leakage, no gold-order mirror, and no unsupported rubric vocabulary. A few reconstructed rationales are stronger than the plan grounds (noted in scoring as ungrounded reconstruction for F29 and F39), but that is ordinary inference/compression rather than clear contamination.
