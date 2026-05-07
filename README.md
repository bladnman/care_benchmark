# Benchmark Instance — CARE

This repo is a self-contained instance of the CARE benchmark. To run it, you'll invoke an AI agent inside the repo (Claude Code, Cursor CLI, etc.) and tell it how many runs to do and which models to use. The agent (acting as **orchestrator** per [AGENTS.md](AGENTS.md)) handles everything else — atomically claiming a wave folder, spawning parallel subagents to plan and evaluate, writing reports.

## Quick run

**End-to-end (one orchestrator call, three phases inside it):**

```
> Run 5 plans on Opus 4.7 very-high, then evaluate them with Opus 4.7 very-high.
```

The orchestrator claims `runs/wave_NNN/`, spawns 5 parallel **phase-1 planner** subagents, waits, spawns 5 parallel **phase-2A blind reconstructor** subagents (each reads only its assigned PLAN — no gold list, no rubric, no PRD), runs a mechanical contamination audit, then spawns 5 parallel **phase-2B scorer** subagents (which read frozen RECONSTRUCTIONs + gold + rubric and score, write reports, write a validity audit, write strict-schema SCORES.json).

Phase 2 is split into 2A and 2B to enforce the **validity contract**: the reconstructor must NOT see the gold list before reconstructing. If it did, the benchmark would measure fill-in-the-blanks against a known target rather than intent carry-through. The freeze rule (RECONSTRUCTION.md is frozen before phase 2B starts) guarantees this.

**Phase-by-phase (two orchestrator calls — useful for different harnesses or different time windows for plan vs eval):**

```
Call 1: > Run 5 plans on Opus 4.7 very-high.
Call 2 (later, possibly different harness): > Evaluate the unfinished plans with Opus 4.7 very-high.
```

Phase 2 auto-detects unscored plans in the most recent wave.

**Single run for testing:**

```
> Run 1 plan and evaluate it with Opus 4.7.
```

## Where things land

| Location | Contents |
|---|---|
| `prd/` | The PRD the planner reads (shipped) |
| `phase_two.zip` or `phase_two/` | Evaluation kit — gold whys, rubric, report templates, scores schema (shipped; may be zipped) |
| `runs/wave_NNN/plans/MMM/PLAN.md` | One plan per run |
| `runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json` | Planner runtime metadata |
| `runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md` | Phase-2A reconstructor's plan-derived intent reconstruction (frozen after write — never modified) |
| `runs/wave_NNN/scoring/MMM/REPORT.md` | Phase-2B scored report (markdown, full detail) |
| `runs/wave_NNN/scoring/MMM/REPORT.html` | Phase-2B scored report (interactive HTML, self-contained) |
| `runs/wave_NNN/scoring/MMM/VALIDITY_AUDIT.md` | Phase-2B contamination audit of the frozen reconstruction (PASS / FLAG / FAIL) |
| `runs/wave_NNN/scores/run_MMM.json` | Strict-schema, pollution-safe scores (consolidator-readable) |
| `runs/wave_NNN/METADATA.json` | Wave-level metadata (started_at, completed_at, defaults, audit warnings) |

## Multi-run guarantees

- **Multiple waves coexist.** `wave_001/`, `wave_002/`, etc. Each is fully self-contained.
- **Concurrent invocations are safe.** Each orchestrator atomically claims a different wave number via `mkdir`. No race.
- **Old waves stay in `runs/`.** They're the historical record. Compress or delete manually if you want to declutter.
- **Within a wave, parallel subagents stay in their assigned slots** (allowlist-enforced + post-hoc audit).

## Variables tracked per run

Each run's `SCORES.json` captures, per side (candidate and evaluator):

- `model`, `effort_level`, `temperature`, `harness`, `provider`
- `parameters` — extension bag for additional bounded scalar metadata (seed, prompt variant, etc.)

Plus run number, optional run label, real ISO 8601 timestamp, and the strict-schema scores. See `phase_two/SCORES_SCHEMA.md` for the full schema.

## Why a strict schema for SCORES.json?

The score files sit loose in `runs/wave_NNN/scores/`. By design, even if a future planner subagent read one of these files (against instructions), nothing in it could prime or bias the planner — only numbers and bounded metadata. All qualitative content (per-feature recovery, per-why narrative, multi-layer breakdowns) lives **only** in `REPORT.md` and `REPORT.html`.

## Underneath

For the workflow logic the orchestrator runs, see [AGENTS.md](AGENTS.md). For the substantive planning task each phase-1 subagent does, see `prd/1-START_HERE.md`. For the substantive scoring procedure each phase-2 subagent applies, see `phase_two/RUBRIC.md` (and `phase_two/2-START_HERE.md` for the operational shape).

## What `.gitignore` is for

Repo hygiene only — keeps result folders out of source-controlled commits. **It is NOT an isolation mechanism.** Phase isolation and pollution-safety are enforced by the orchestrator's prompt allowlists, the post-hoc audit step, and the strict SCORES.json schema. Not by gitignore.
