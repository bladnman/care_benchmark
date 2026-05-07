# Benchmark Instance - CARE

This repo is a self-contained CARE benchmark instance. It is deliberately split so phase-1 planning contexts only receive phase-1-safe instructions.

## Entry Points

- For plan generation / phase 1: read `PHASE_ONE_INSTRUCTIONS.md`.
- For post-plan / later-phase work: expand `phase_two.zip`, then read `phase_two/PHASE_TWO_INSTRUCTIONS.md`.
- For harness routing: `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md` are intentionally tiny dispatch files.

The repo intentionally does not ship harness-specific agent definitions such as `.cursor/agents/`. Use the phase instruction files with whichever runner or harness is orchestrating the wave.

## Files

| Location | Contents |
|---|---|
| `prd/` | Product requirements read by phase-1 planners |
| `PHASE_ONE_INSTRUCTIONS.md` | Phase-1 orchestrator workflow and planner prompt |
| `phase_two.zip` | Later-phase instruction package |
| `runs/wave_NNN/` | Generated benchmark wave output |

## Hygiene

`runs/`, unzipped `phase_two/`, and `.cursor/` are ignored repo-local state. Ignore rules are for source-control hygiene only; phase separation is enforced by the active phase instructions and post-run audits.
