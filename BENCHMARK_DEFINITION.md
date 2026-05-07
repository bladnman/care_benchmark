# CARE — Benchmark Definition

**CARE = Capture And Recovery Eval.**
**What it measures:** how much intent (ask + why) survives the encoding step from a rich PRD into a structured plan — i.e., *intent carry-through*.
**Status:** v02.1 (current).
**Working titles during design:** `two_score_inversion`, then `intent_carry_through`. Both refer to this same test under earlier codenames.
**Last update:** 2026-05-07

---

## 1. What this measures

How much **intent** (the ask + the why) survives the encoding step from a rich requirements document (PRD) into a structured plan. The PRD-to-plan transformation is lossy by nature; CARE measures *how lossy*, model by model.

Motivating observation: planners — even strong ones — can saturate on "did the plan address the requirement?" rubrics while quietly compressing the *why* behind each decision into a label. The plan looks complete but is a poor handoff to a downstream builder. CARE surfaces that gap explicitly.

CARE is a successor concept to the existing [planning benchmark](https://github.com/bladnman/planning_benchmark), which has approached saturation on capture. CARE keeps capture as a precondition and adds a second axis — *did the why survive into the plan* — that current frontier models do not yet saturate.

---

## 2. Definition of intent

**Intent = ask + why.** Two flavors of why, both count:

- **Functional / correctness whys.** If dropped, the system is built *wrong*.
  > Example: *"Use synthetic_id (not user_id) for shard partitioning, because user_id is email-derived PII and would leak into Kafka partition metadata."*
  > Drop the why and a builder agent might fall back to user_id; the system technically works but produces an audit/compliance failure.

- **Experiential / affective whys.** If dropped, the system *feels wrong*.
  > Example: *"Animate the send button while the user is typing — so the user feels the system is listening, that it's alive and cares, not dead."*
  > Drop the why and a builder might pick a robotic strobe instead of a heartbeat-like pulse; both satisfy the literal ask, only one is right.

The benchmark covers both. The PRD's gold-why list mixes the two on purpose; affective whys are typically the leakier ones.

---

## 3. Whys have anchors — system-level or feature-level

Every gold why has an **anchor**. Two anchor types, both first-class:

1. **System-level whys.** Cross-cutting design intent — the philosophy of the product. Written once in the PRD (typically in product-brief / design-principles / voice-and-tone sections) and *inherited* by many features without being restated each time.
   > Example: *"The product should feel alive — animations should be breathy and personality-driven, not robotic. Users should feel the system is listening, not announcing."* This intent shapes dozens of features (greetings, transitions, idle motion, input feedback) without being attached to any one of them.

2. **Feature-level whys.** Bound to a specific named feature in the PRD. Two flavors:
   - **Exceptions to system-level intent.** *"This delete confirmation button is NOT bird-shaped — delete should feel grave, not playful."* The exception itself is the why.
   - **Feature-specific intent that goes beyond the system level.** *"Bird greeting on return uses a stagger because we want the user to feel seen, not announced at."* Not an exception — a sharpening.

**What we do NOT have:** a forced "every feature carries a gold why" rule. Most features in the PRD silently inherit system-level intent and carry no feature-level why. That mirrors how real PRDs are written and avoids filler whys ("we use button-shaped buttons because users expect button-shaped affordances") that would be easy to game.

**What we also do NOT have:** floating whys with no anchor. Every gold why points either to a system-level concept (named in the PRD) or to a specific feature.

**Weight tiers (1 / 2 / 4) apply to both kinds.** Some whys matter much more than others when dropped:

- **Weight 1** — real but subtle backstory; nice to preserve, not catastrophic if lost.
- **Weight 2** — contentful rationale; planner should carry through.
- **Weight 3** — load-bearing. Drop and the wrong system gets built (or feels wrong in a way the user will notice).

Most system-level whys naturally land at weight-2 or weight-3 (a system-level principle that matters to nothing isn't really a system-level principle). Feature-level whys distribute across all three tiers, with most feature-level whys being weight-2 or weight-3 (the trivial ones don't earn an explicit gold why; they inherit from the system level).

Implication for scoring:
- **System-level whys** are scored at the plan level: *"Did this design principle survive as a cross-cutting concern in the plan?"* Yes / partial / no, weighted.
- **Feature-level whys** are scored per-feature: *"Did this captured feature carry its bound why through?"* Yes / partial / no, weighted.

Both contribute to intent fidelity (see §5).

---

## 4. Execution

Two subagents run in **separate sessions, fresh context, zero prior exposure to each other's intermediate state**:

1. **Planner A** receives the PRD. Produces `PLAN.md` — a structured plan covering the PRD.
2. **Reconstructor B** receives only `PLAN.md` (never the PRD). B does two passes:
   - **System-level pass.** Walk the plan; identify any cross-cutting design principles, philosophies, or product-voice intent the plan carries. Write them down with enough detail that a scorer can map them to the gold system-level whys.
   - **Feature-level pass.** For each feature B can identify in the plan, attempt to reconstruct the why bound to that feature. If a feature's why is not recoverable from the plan alone, B marks `NOT RECOVERABLE FROM PLAN` rather than confabulate.

A and B are scored against the same held-out gold list:

- **Planning quality** = (features A captured) ÷ (total features in PRD)
- **Intent fidelity** = `Σ(weight × recovery-score)` over all gold whys ÷ `Σ(weight)` over all gold whys *whose anchor is reachable*. A gold why's anchor is reachable if it is a system-level why (always reachable; the plan exists as a whole) or if it is a feature-level why bound to a feature A captured (otherwise the feature is gone from the plan and there's no place its why could land).

Both numerator and denominator of fidelity are weight-summed (each gold why contributes its weight tier — 1, 2, or 4 — rather than counting one each). This way the score reflects load-bearing-ness: a model that recovers all weight-3 whys but misses weight-1s scores higher than one losing across the board.

Planning quality and intent fidelity are independent measurements. Intent fidelity for feature-level whys is conditional on capture (denominator only counts feature-level whys whose feature A captured); intent fidelity for system-level whys is unconditional (the plan always has an opportunity to preserve a system-level principle).

### Operational requirements

- **Real fresh context** between A and B. Different processes / different sessions / no shared conversation. The "same model in the same conversation, told to ignore what came before" approximation is *not* sufficient — it leaks. The whole rubric integrity sits on this.
- **Plan is the only bridging artifact** between A and B. No web search, no rubric exposure, no PRD glimpse for B.
- **Reconstruction must be honest about non-recovery.** B is instructed to mark `NOT RECOVERABLE FROM PLAN` for items it can't justify from the plan alone. Sycophancy / confabulation is a known failure mode; it must be guarded against in the prompt.
- **Reconstructor's two passes must be done in one pass over the plan.** B is asked for system-level intent and per-feature whys in the same RECONSTRUCTION.md output (separate sections). One pass over the plan, two output sections. This avoids re-reading bias.

---

## 5. Scoring

Three numbers per model run:

| Score | Definition | Range | What it discriminates |
|---|---|---|---|
| **Planning quality** | features captured ÷ total | 0–100% | Capacity to plan everything asked |
| **Intent fidelity** | weighted whys recovered ÷ weighted whys reachable | 0–100% | Quality of the encoding step itself |
| **Combined quality** | `planning% × 100 + fidelity% − 100` | 0–10,000 | Lexicographic rank for leaderboard |

The combined score is **for ranking only**, not for charting:

- Planning quality dominates the rank. Each 1pp of planning is its own 100-point band on the combined score.
- Intent fidelity slides you within the band by up to 99 points.
- A 90% planning model with worst fidelity (8,900) ties with an 89% planning model at perfect fidelity (8,900) — boundary-tied but never inverted.
- 100% / 100% = 10,000 (perfect).

### Diagnostic split: system-level vs feature-level fidelity

The single fidelity number is the headline. The report should also break it into two sub-components for diagnostic value:

- **System-level fidelity** = `Σ(weight × recovery-score)` over system-level gold whys ÷ `Σ(weight)` over system-level gold whys.
- **Feature-level fidelity** = `Σ(weight × recovery-score)` over feature-level gold whys whose feature A captured ÷ `Σ(weight)` over those same whys.

A model that preserves the philosophy but loses per-feature specifics looks different from one that captures specifics but flattens the philosophy into "design principles: see PRD." Both are interesting failure modes; the split surfaces them.

### Reading a combined score at a glance

`8,863` reads as "in the 89% planning band, 37 points of fidelity loss within the band." The integer part of `score / 100` ≈ planning percentage; the two-digit remainder ≈ fidelity adjustment. Diagnostic info baked into the digits.

### Why a single combined chart is wrong

On a 0–10,000 axis, a 13-point fidelity difference between two models in the same planning band is 0.13% of the y-range — invisible. The combined score's lexicographic structure mathematically guarantees fidelity is dwarfed by planning. That's the right behavior for ranking, the wrong behavior for visualization.

### What to chart instead

**A 2D scatter of (planning quality, intent fidelity).** Both axes 0–100. Each model is a dot. Upper-right is best.

Cluster shapes across the model population reveal:
- Frontier of dots along `y = x` → "all models trade off planning for fidelity equally"
- Flat front at high planning, varying fidelity → "planning is solved, fidelity is the open problem"
- Dense clusters at specific (P, F) pairs → "this generation hit a plateau"

Trellis plots (small multiples by planning tier) are an option if a single combined-score chart is required for some specific use case.

---

## 6. Applicability range

This test is for models that can plan competently. **Below a planning-quality floor, intent fidelity readings are degenerate** — a model that captured 1 feature with 1 perfect why scores 100% fidelity on n = 1: technically correct, statistically meaningless.

Conventions:

- Report all three numbers always (planning, fidelity, combined).
- Below ~30% planning: flag fidelity as low-confidence or omit from leaderboards.
- For absolute filtering, pre-screen with a planning-competence check before fidelity reporting.

This is a feature, not a bug. The benchmark specializes in the encoding step *for models that can plan*. A separate test handles "can this model plan at all."

---

## 7. Hardening levers (without tricking)

User intent: *"I don't want to trick anything, but I do want to make it difficult."*

**Levers we use:**

- **Two anchor types — system-level + feature-level whys** (see §3). Mirrors how real PRDs are written: cross-cutting design intent + feature-specific exceptions and sharpenings. Eliminates filler whys (we don't force a why on every feature) without going sparse (system-level whys cover huge swaths via inheritance).
- **Asymmetric weighting by load-bearing-ness.** Weight tiers 1 / 2 / 4. Already baked into the scoring per §5; it's the lever that lets the test reflect which whys actually matter.
- **Multi-layer whys.** On weight-3 whys (system-level *or* feature-level), the gold why decomposes into primary cause + secondary contributor + downstream consequence. Score recovery of all three layers separately; the chain becomes the unit, not a single fact.
- **N reconstruction attempts at temperature.** Variance signal; the static one-shot run misses it. Useful once v1 is stable.
- **PRD complexity.** More domain nuance; more easily-confused distractors; more surface area for the planner to compress incorrectly.
- **Mix of why types.** ~50% functional, ~50% affective across all weight tiers and both anchor types. Affective whys typically leak more, especially when they live at the system level (where the planner is most tempted to compress them into "design principles: see PRD").

**What we avoid (would feel like tricking):**

- Steganographic hiding of intent.
- Mandatory plan style (forces the planner into a specific format that isn't really about intent).
- Obscure jargon that's hard to recognize as binding.
- Whys that aren't actually load-bearing in the system being described.

---

## 8. Harness recommendation

Develop the test in **one harness first**. Add a second harness as a generalization check **once the result is stable across N runs**.

Reasoning: harness-induced variance (subagent construction, rubric-exposure norms, tool-surface differences) can plausibly add ±10 points of noise — enough to swamp a 30-point gap if introduced too early.

Once the gap is reliable in harness A, run a single replication in harness B. If the gap survives, it's a model property. If it doesn't, you've learned something about your test, not your models.

---

## 9. Quiet assumptions worth flagging

These are baked into the test design and may be wrong:

- **"Fresh context" between A and B is operationally clean.** Hardest assumption to satisfy in practice. See §4.
- **Plan is the binding artifact.** In real workflows, intent might land in the runbook, ADR, or PR description instead. This test scopes intent durability through one specific artifact type.
- **B will honestly mark NOT RECOVERABLE.** Sycophancy risk; B may confabulate plausible whys. Mitigate via prompt + spot-check.
- **Gold whys are the only whys that matter.** False negatives possible if A captured intent in a way we didn't gold-label.
- **"Intent" is the right primitive.** Maybe it's "load-bearing rationale" or "constraint" — slightly different concepts. We're choosing intent (ask + why, both kinds) for v1.

---

## 10. What this doesn't measure

- Capture-discrimination on simple PRDs (planning quality saturates on frontier models — that's a different test, kept as a complement).
- System correctness once built (this is a planning-stage test, not a build-stage test).
- Per-feature implementation quality (the plan is the artifact under test, not the eventual code).

---

## 11. Open questions

- **Volume of gold whys.** First instance: ~120 features, gold whys distributed across system-level + feature-level anchors as the architect judges natural for the product. Best-guess shape: ~5–15 system-level whys + ~25–50 feature-level whys = ~30–65 gold whys total. Empirical whether that's the right density.
- **System-level / feature-level mix.** What ratio? V1 lets the architect decide what feels natural for Pocket Aviary; v2 may want a target ratio.
- **Weight-tier distribution.** With filler whys removed, the natural distribution likely skews more toward weight-2 and weight-3. Open whether to target a distribution or let it emerge.
- **Semantic vs keyword scoring of reconstruction.** A reconstruction that captures the why in different words must still count. Rubric needs explicit semantic-equivalence judgment.
- **System-level scoring granularity.** Score per principle as a single yes/partial/no? Or score per principle *as preserved across the features that should have inherited it*? V1 takes the simpler approach (per principle); v2 may revisit.
- **Treatment of low-capture / high-fidelity models.** Currently: report all three numbers, mark low-confidence below the floor. May need stricter gating.
- **Functional vs affective why weighting.** Equal for v1; could shift if data warrants.
- **Multi-layer whys.** Used on weight-3 whys for v1 (both anchor types); whether multi-layer adds signal or noise is empirical.

---

## 12. Related artifacts

> **Note:** this is a copy of the spec bundled inside a shipped benchmark-instance repo. The "Related artifacts" referenced below live in the design project that produced the instance, not in this checkout. The shipped instance is self-contained — `prd/`, `phase_two/`, `AGENTS.md`, `runner_template` outputs in `runs/` — and does not need the build-process artifacts to operate.
>
> If you need the design-process artifacts (build prompts, prior runs, history), find them in the original design project:
>
> - `run_001/saturation/` — original v1 prototype instance and findings (precursor to this build).
> - `run_002/PROMPT_team_orchestration.md` — agent-team prompt that builds and tests a v1 instance per this definition (under the working title `intent_carry_through`).
> - `run_002/` — full build outputs (architect, prd, annotation, critique, test, report).

---

## 13. History

- **2026-05-04** — Original `two_score_inversion` candidate built and tested by the saturation tester. Reported a 33-point gap between capture (89%) and reconstruction (56%) on Opus 4.7. PASS-PARTIAL with bias caveat (rubric-exposed self-run).
- **2026-05-05** — Clean fresh-context rerun. Same triple. Candidate marked the user's #1 favorite in the run_001 review (favorite=3, interesting=3).
- **2026-05-05** — Conversation between user and Claude expanded the framing significantly: dual-score split (planning quality + intent fidelity); combined score lexicographic on planning; 2D scatter for visualization; intent definition split into functional + affective; whys-bound-to-features rule; applicability range; hardening levers; harness-development recommendation. Candidate renamed `two_score_inversion` → `intent_carry_through`.
- **2026-05-05** — Spec codified (this document). Run-002 orchestration prompt drafted; agent team to build the v1 PRD instance and produce a first measurement.
- **2026-05-05** — Design correction: every feature carries a gold why, with weight tiers 1/2/4 (not the prior 1/5-of-features-have-whys hedge). Reasons: no architect-selection bias, ~5× denser fidelity signal, and matches the user's "a feature is the ask + the why" model of intent. SPEC §3 and §7 updated; orchestration prompt updated to match.
- **2026-05-06** — Design correction (supersedes the 2026-05-05 entry above): the "every feature carries a gold why" rule is replaced with a two-anchor model — **system-level whys** (cross-cutting design intent, written once, inherited by many features) + **feature-level whys** (bound to a specific feature, typically an exception to or sharpening of system-level intent). Most features in the PRD now silently inherit system-level whys and carry no feature-level why. Reason: per-feature filler whys risked making the score gameable (a planner that mentions generic platitudes earns credit) and didn't match how real PRDs are written. The two-anchor model preserves signal density (system-level whys cover broad surface area via inheritance), eliminates filler authoring pressure, and produces a more realistic test substrate. SPEC §3, §4, §5, §7, §11 updated; orchestration prompt §2, §4, §5 (Phases A–F) updated to match.
- **2026-05-07** — Validity bug found in single-phase-2 procedure: a single evaluator subagent reading both PLAN and gold/rubric in the same context produced fill-in-the-blanks reconstructions and inflated scores. Fix: phase 2 split into 2A (blind reconstruction; reads PLAN only) and 2B (scoring; reads PLAN + frozen RECONSTRUCTION + gold + rubric, can't modify reconstruction). Freeze rule is the validity contract. Runner v02.1 implements this.
- **2026-05-07** — Renamed from working title `intent_carry_through` to **CARE** (Capture And Recovery Eval). The property under test is still "intent carry-through" (does intent carry through the encoding from PRD to plan?); CARE is the benchmark's name.
