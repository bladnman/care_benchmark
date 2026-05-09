# VALIDITY_AUDIT — CARE run 001

## 1. ID Leakage Check

No gold IDs or rubric IDs were found in the frozen reconstruction. It does not use `S1`-`S9`, `F1`-`F40`, `R-F01`, or similar external taxonomy. The reconstruction uses plan-derived labels such as `Scope and Strategy`, `Architecture`, `Data Model`, and `Simulation Engine Design`, plus the phase-2A marker `NOT RECOVERABLE FROM PLAN`. No leakage hit.

## 2. Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| `long-term relationships with procedural birds through observation and idle attention` | PLAN §1 uses the same phrase. | Plan-derived. |
| `persistent aliveness independent of the user's presence` | PLAN §2 says the architecture ensures aliveness independent of presence. | Plan-derived. |
| `absolute prohibition on streaks, levels, achievements, or scores` | PLAN §1.2 says this directly. | Plan-derived. |
| `read-only for state; write-only for events` | PLAN §6.1 says this directly. | Plan-derived. |
| `naturalist prose descriptions of the scene every 45s` | PLAN §7.1 says narration updates every 45s with naturalist prose. | Plan-derived. |
| `PII isolation` | PLAN §6.2 uses `PII Isolation`. | Plan-derived. |
| `NOT RECOVERABLE FROM PLAN` | Not a plan phrase; it is phase-2A reconstruction protocol wording. | Not a gold/rubric leakage concern by itself. |

I did not see rubric-side terms such as `feature-level fidelity`, `weight-3`, `multi-layer recovery`, `intent fidelity`, or gold IDs used freely in the reconstruction.

## 3. Heading Mirror Check

The reconstruction headings mirror the PLAN, not the gold list. After `## System-level intent` and `## Per-feature whys`, the lower headings are `Scope and Strategy`, `Out-of-Scope / Non-Goals`, `Architecture`, `Data Model`, `Simulation Engine Design`, `Audio Pipeline`, `Sync and Privacy`, `Accessibility`, `Performance and Observability`, and `Rollout and Risks`. These match PLAN section names and do not mirror `GOLD_WHYS.md` headings such as system-level IDs S1-S9 or feature-level IDs F1-F40.

## 4. 1:1 Mapping Suspect Check

No 1:1 gold mapping pattern is present. The reconstruction is organized by candidate plan sections and plan bullets, not by the 9 system whys and 40 feature whys. It includes many plan-derived implementation items that are not gold-why rows, and it omits or marks not recoverable several gold targets. This is not a neat gold-order reconstruction.

## 5. Plan-Derivation Spot Check

1. Reconstruction: `Persistent aliveness independent of the user's presence.`
   PLAN support: §2 says the system follows a thin-client, thick-server pattern to ensure aliveness independent of the user's presence.
   Result: supported.

2. Reconstruction: `No punishment for absence.`
   PLAN support: §1.2 says no hunger, death, distress meters and no punishment for absence; §4.2 says neglect results in ambient stagnation with no negative drift.
   Result: supported.

3. Reconstruction: `Narration provides an aria-live region with naturalist prose descriptions of the scene every 45s.`
   PLAN support: §7.1 says the aria-live region is updated with naturalist prose descriptions every 45s.
   Result: supported.

## 6. Verdict

**PASS.** The reconstruction reads as plan-derived: no gold IDs, no gold-order 1:1 mapping, headings follow the PLAN, and sampled articulate sentences are supported by PLAN passages. The only non-plan phrase of note is `NOT RECOVERABLE FROM PLAN`, which is expected reconstruction protocol rather than contamination.
