# VALIDITY_AUDIT - CARE run 001

## ID Leakage

Verdict: no gold ID leakage found. A search of the frozen reconstruction for gold IDs and rubric-side labels found no S1-S9, F1-F40, R-Fxx, gold, rubric, feature-level fidelity, weight-3, or multi-layer references. The only suspicious-looking term was "load-bearing" in RECONSTRUCTION.md lines 3, 7, and 9, but PLAN.md has a "Load-Bearing Decisions from PRD" heading at line 767, so this is plan-derived.

## Vocabulary Check

| Reconstruction phrase | PLAN support | Assessment |
|---|---|---|
| "Server-canonical life, rendered locally" | PLAN lines 73-85, 118-133, 382-389 | Derived from canonical-state/server-render boundary. |
| "Additive, ordered change instead of overwrites" | PLAN lines 391-400 | Directly plan-derived from additive deltas and no LWW. |
| "Slow, monotonic personality change over weeks" | PLAN lines 337-343, 356-361 | Directly plan-derived. |
| "relationship shape" | PLAN line 772 | Exact plan phrase. |
| "Naturalist product voice over exposed mechanics" | PLAN lines 559-562, 741-743 | Derived from naturalist prose and no personality numbers. |
| "Accessibility as a designed surface" | PLAN lines 571-581, 718-723 | Directly plan-derived. |
| "Performance as part of the spell" | PLAN lines 725-733, 797-803; "spell" also appears at line 706 | Plan-derived phrasing, not rubric language. |
| "Restrained social and telemetry boundaries" | PLAN lines 22, 28-32, 641-644 | Plan-derived synthesis. |

No rubric-only terms were used freely. The mandated headings "System-level intent" and "Per-feature whys" come from the phase-2A output format, not from gold leakage.

## Heading Mirror

The reconstruction contains only two Markdown headings:

| Reconstruction heading | Gold-list similarity | Assessment |
|---|---|---|
| System-level intent | Generic phase-2A required section, not a gold title | Not suspicious. |
| Per-feature whys | Generic phase-2A required section, not a gold title | Not suspicious. |

There are no heading-by-heading mirrors of the gold S1-S9 or F1-F40 lists.

## 1:1 Mapping Suspect

No 1:1 gold mapping was observed. The reconstruction is organized by the plan's own sections: V1 scope, out-of-scope, architecture, simulation engine, frontend rendering, audio, accessibility, performance, and rollout. It does not list S1-S9 or F1-F40, does not preserve gold order, and includes plan-specific items such as "Tick cadence: NOT RECOVERABLE FROM PLAN," "Top bar fade," and rollout feature flags. This reads as plan-derived rather than gold-derived.

## Plan-Derivation Spot Check

1. Reconstruction line 5 says additive deltas let laptop and phone contributions apply "in event-log order." PLAN lines 393-400 explicitly describe no last-write-wins, additive server-authored deltas, and the laptop/phone conflict example.
2. Reconstruction line 13 says reduced motion is "Not 'animations off'" but "a designed surface" and that accessibility ships with v1. PLAN lines 571-581 and 718-723 contain those points directly.
3. Reconstruction line 97 says presence counts only when visible, focused, and recently active, avoiding tab-open drift corruption. PLAN lines 404-410 and 745-753 give the strict conjunction and active-usage-not-tab-open mitigation.

All three checked claims are supported by the assigned PLAN.

## Verdict

PASS. The reconstruction shows no gold-ID leakage, no rubric-vocabulary contamination, no gold-heading mirror, and no neat 1:1 mapping to the gold list. It reads as a plan-derived synthesis with cautious NOT RECOVERABLE calls, and the most articulate claims spot-check cleanly against PLAN.md.
