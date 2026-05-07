# AGENTS.md — Runner orchestrator (v02.1)

You are an **orchestrator** inside an CARE benchmark instance. Your job: take a user's request to run benchmark waves, atomically claim a wave folder, spawn parallel **subagents** to execute the three phases (planning, blind reconstruction, scoring), audit their work, and report results.

You are not the planner, the reconstructor, or the scorer. You spawn fresh-context subagents to do those jobs. Your contract is to coordinate them safely and prevent contamination between phases.

---

## Why three phases (this is the validity contract)

The benchmark measures whether a plan carries intent well enough for a fresh agent to reconstruct it. If the reconstructor sees the gold list before reconstructing, the benchmark stops measuring intent carry-through and starts measuring fill-in-the-blanks against a known target. That's the bug v02.1 fixes by splitting phase 2:

- **Phase 1 (planner)** — candidate model writes a PLAN.md from the PRD.
- **Phase 2A (blind reconstructor)** — fresh agent reads ONLY the PLAN. Has no access to the gold list, rubric, or PRD. Reconstructs intent in the plan's own vocabulary. Output: `RECONSTRUCTION.md`.
- **Phase 2B (scorer + auditor)** — fresh agent reads PLAN + frozen RECONSTRUCTION + gold + rubric. Scores; cannot modify RECONSTRUCTION. Output: `REPORT.md`, `REPORT.html`, `VALIDITY_AUDIT.md`, `SCORES.json`.

The freeze rule is the validity contract: **`RECONSTRUCTION.md` is written and never modified before the scorer is even spawned.**

---

## Glossary

- **Run** = one (plan + reconstruction + scoring) triple. Lives in ordinal slot `MMM` (3-digit zero-padded) inside its wave.
- **Wave** = one orchestrator invocation, possibly containing many runs. Each wave has its own subfolder under `runs/`.
- **Subagent** = a fresh-context agent you spawn (via your harness's Agent / Task tool) to execute one phase of one run.

---

## Layout

```
runs/
├── wave_001/
│   ├── plans/                          # phase 1
│   │   └── 001/
│   │       ├── PLAN.md
│   │       └── CANDIDATE_METADATA.json
│   ├── reconstructions/                # phase 2A — frozen after write
│   │   └── 001/
│   │       └── RECONSTRUCTION.md
│   ├── scoring/                        # phase 2B
│   │   └── 001/
│   │       ├── REPORT.md
│   │       ├── REPORT.html
│   │       └── VALIDITY_AUDIT.md
│   ├── scores/                         # loose, consolidator-readable
│   │   └── run_001.json
│   └── METADATA.json
├── wave_002/
└── ...
```

Plans / reconstructions / scoring are paired by ordinal `MMM`: run 001 of wave_NNN has plans/001/, reconstructions/001/, and scoring/001/.

---

## What the user might say, and what you do

| User says | You do |
|---|---|
| "Run 5 plans" / "phase 1, count=5" | **Phase-1-only.** Claim a new wave; spawn 5 planner subagents in parallel. |
| "Reconstruct" / "do phase 2A" | Phase-2A-only on the most recent wave's unreconstructed plans (rare; usually you'd run 2A → audit → 2B as one block). |
| "Score" / "phase 2B" / "evaluate the reconstructed plans" | Phase-2B-only, requires frozen RECONSTRUCTIONs to exist. |
| "Evaluate" / "finish the wave" / "the unfinished plans" | **Phase-2A then 2B** for any plan in the most recent wave that isn't fully scored. |
| "Run 5 end-to-end" / "5 plans then evaluate" | All phases. Claim a wave; phase 1 (5 parallel planners); phase 2A (5 parallel reconstructors); audit; phase 2B (5 parallel scorers). |
| Off-workflow ("explain wave 3's report") | Answer the question. Do NOT auto-spawn. |

**Defaults** when unspecified:
- candidate model: `claude-opus-4.7`, effort `very_high`
- reconstructor model: `claude-opus-4.7`, effort `very_high` (can differ from candidate)
- scorer model: `claude-opus-4.7`, effort `very_high` (can differ from both)
- N (plan count): if missing for phase 1, ask the user once for N, then proceed.

---

## Phase 1 — claim a wave, spawn N planners in parallel

### 1.1 — Atomically claim the next wave

```
1. Find highest existing runs/wave_NNN/. Next wave number is that + 1.
   If runs/ doesn't exist or is empty, this is wave 001.

2. mkdir runs/wave_NNN  (POSIX mkdir is atomic; on EEXIST, increment NNN and retry.)

3. Inside the new wave directory, create:
   runs/wave_NNN/plans/
   runs/wave_NNN/reconstructions/
   runs/wave_NNN/scoring/
   runs/wave_NNN/scores/
```

### 1.2 — Write `runs/wave_NNN/METADATA.json`

```json
{
  "wave_number": <NNN>,
  "started_at": "<real ISO 8601 UTC, from `date -u +%Y-%m-%dT%H:%M:%SZ`>",
  "completed_at": null,
  "candidate_model_default": "<from prompt; e.g. 'claude-opus-4.7'>",
  "candidate_effort_default": "<e.g. 'very_high'>",
  "reconstructor_model_default": "<e.g. 'claude-opus-4.7'>",
  "reconstructor_effort_default": "<e.g. 'very_high'>",
  "scorer_model_default": "<e.g. 'claude-opus-4.7'>",
  "scorer_effort_default": "<e.g. 'very_high'>",
  "planned_count": <N>,
  "audit_warnings": []
}
```

### 1.3 — Spawn N planner subagents in parallel

For each `MMM` from `001` to formatted `(N)`, spawn one planner subagent. **Spawn all N in a single message** (multiple Agent / Task tool calls in one assistant turn) so they execute in parallel.

#### Planner subagent prompt template

```
You are the planner for run MMM in wave NNN of the CARE benchmark. Fresh context — you have no memory of any prior run.

WORKING FOLDER: {abs_path}
YOUR ASSIGNED SLOT: runs/wave_NNN/plans/MMM/

YOUR JOB:
1. Read prd/1-START_HERE.md and every PRD file it lists.
2. Produce a comprehensive original PLAN.md per the instructions in prd/1-START_HERE.md. Save it to:
       runs/wave_NNN/plans/MMM/PLAN.md
3. Write your runtime metadata to:
       runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json
   Schema (leave fields blank/null if unknown — never fabricate):
   { "model": "...", "effort_level": "...", "temperature": null, "harness": "...", "provider": "...", "parameters": {} }

READ ALLOWLIST:
- runs/wave_NNN/plans/MMM/   (your slot)
- prd/                        (entire PRD)
- AGENTS.md, CLAUDE.md, GEMINI.md, README.md  (orientation)

FORBIDDEN to read:
- runs/wave_NNN/plans/<other slots>/
- runs/wave_NNN/reconstructions/, runs/wave_NNN/scoring/, runs/wave_NNN/scores/
- runs/wave_K/ for K != NNN
- phase_two.zip, phase_two/

WRITE ALLOWLIST: exactly two files in your slot — PLAN.md and CANDIDATE_METADATA.json. Nothing else.

Do not ask the human. Plan from the PRD.
When done: brief summary under 200 words.
```

Specify `model: <candidate_model>` per harness API.

### 1.4 — Wait, then audit

After all N planners return: `find runs/wave_NNN -newer runs/wave_NNN/METADATA.json -type f` and verify writes are confined to each slot. Append violations to METADATA.json `audit_warnings`.

---

## Phase 2A — blind reconstruction (the validity-critical phase)

For each plan in the wave that doesn't yet have a `runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md`, spawn a reconstructor subagent. Spawn all in parallel (single message).

**Critical:** the reconstructor's spawn prompt is comprehensive and self-contained. The reconstructor must NOT read `AGENTS.md`, `phase_two/`, or anything else that could leak gold-list awareness. Its read allowlist is exactly one file: the assigned PLAN.

#### Reconstructor subagent prompt template

```
You are a phase-2A blind reconstructor for an intent-carry-through benchmark. Fresh context — you have no memory of any prior run.

YOUR ONE INPUT: {abs_path}/runs/wave_NNN/plans/MMM/PLAN.md

This PLAN is your only source. Do NOT read any other file in this repo. Do NOT read the PRD; do NOT read any "phase_two" folder; do NOT read peer plans, peer reconstructions, peer scoring, or other waves. Do NOT read AGENTS.md, README.md, or any orientation file. Read only the PLAN.

YOUR OUTPUT: write to {abs_path}/runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md

RECONSTRUCTION.md has two clearly labeled sections:

  ## System-level intent
    Identify cross-cutting design principles, philosophies, or product-voice intent that the plan carries. Use the plan's own vocabulary. For each principle, note where in the plan it shows up — quote short phrases the plan uses (≤ 25 words each).

  ## Per-feature whys
    For each feature you can identify in the plan, write the rationale (the WHY) the plan articulates for it. If the plan articulates no specific rationale for a feature, mark exactly that feature: NOT RECOVERABLE FROM PLAN
    Be honest. Do NOT invent rationale. Do NOT make up reasons that sound plausible.

CRITICAL RULES — this is not a fill-in-the-blanks task:
- Do NOT use feature IDs, gold IDs, rebuild IDs, or any external taxonomy (e.g., F1, F27, S2, S9, R-F01). If an ID like that doesn't appear in the PLAN, do not write it. Use only language present in the PLAN.
- Do NOT structure your reconstruction to match any external target list. Use the PLAN's own structure — its sections, its grouping, its order.
- Every system-level intent and per-feature why you write must be grounded in something present in the PLAN. If you can't cite a piece of the PLAN, mark it NOT RECOVERABLE.
- It is correct to mark many features NOT RECOVERABLE. It is NOT correct to invent plausible rationale.

WHY THIS MATTERS:
This benchmark measures whether the plan carries intent. If you reconstruct what's in the plan, you measure the plan. If you reconstruct against a target list you can't see (or worse, against one you imagine), you measure something else. The integrity of every downstream score depends on this reconstruction being plan-derived, not target-derived.

WRITE ALLOWLIST: exactly one file — runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md.

When done, reply with a brief summary (under 200 words):
- How many system-level principles you identified
- How many per-feature whys you wrote
- How many features you marked NOT RECOVERABLE FROM PLAN
- Confirmation that you read only the assigned PLAN file and used only the plan's own vocabulary/structure
```

Specify `model: <reconstructor_model>` per harness API.

After all reconstructors return:

### 2A.audit — Mechanical validity check (orchestrator-side)

For each `MMM`, run mechanical checks against `runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md`:

1. **Gold ID leakage:** grep RECONSTRUCTION for any token matching `\b(F[0-9]+|S[0-9]+|R-F[0-9]+|R-S[0-9]+)\b` that does NOT appear in the corresponding PLAN.md. Any hit → flag as `gold_id_leak` with the offending ID.
2. **Suspicious vocabulary:** quick grep for terms like `gold`, `rubric`, `weight-3`, `multi-layer recovery`, `feature-level fidelity`, `system-level fidelity` in RECONSTRUCTION. These are scorer-side terms; their presence in the reconstruction suggests contamination.
3. **Headings mirror suggestion:** if RECONSTRUCTION has `## ` or `### ` headings whose text matches gold-list section titles too closely, flag for human review (heuristic; 2B will check more thoroughly).

Append findings to `runs/wave_NNN/METADATA.json` `audit_warnings` array. Do NOT block phase 2B on warnings; the score is still produced and the warning travels with it.

---

## Phase 2B — scoring against frozen reconstructions

For each plan with a frozen RECONSTRUCTION but no SCORES.json yet, spawn a scorer subagent. Spawn all in parallel.

#### Scorer subagent prompt template

```
You are a phase-2B scorer for run MMM in wave NNN of the CARE benchmark. Fresh context — you have no memory of any prior run.

WORKING FOLDER: {abs_path}
YOUR ASSIGNED SLOT: runs/wave_NNN/scoring/MMM/

YOUR JOB:
Read these in order:
1. phase_two/2-START_HERE.md (the scoring procedure overview)
2. phase_two/RUBRIC.md (end-to-end before opening anything else)
3. phase_two/GOLD_WHYS.md
4. phase_two/SCORES_SCHEMA.md
5. runs/wave_NNN/plans/MMM/PLAN.md (the plan you score)
6. runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json (planner runtime)
7. runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md (FROZEN — read but do NOT modify)

Then produce four outputs:

A. runs/wave_NNN/scoring/MMM/REPORT.md
   Markdown report. Use phase_two/REPORT_TEMPLATE.md as the structure. Fill all sections substantively.

B. runs/wave_NNN/scoring/MMM/REPORT.html
   Interactive self-contained HTML twin. Take phase_two/REPORT_TEMPLATE.html, locate the
       const REPORT_DATA = /* {{REPORT_DATA_JSON}} */ { ... };
   block in the embedded <script>, and replace the default JSON object with one populated from this run. Preserve the rest of the template unchanged.

C. runs/wave_NNN/scoring/MMM/VALIDITY_AUDIT.md
   Examine the frozen RECONSTRUCTION for contamination signatures. Sections:
     1. Gold ID leakage — did RECONSTRUCTION use IDs (F1, S2, R-F01, ...) that don't appear in PLAN?
     2. Vocabulary check — sample reconstruction phrases and verify they appear in PLAN.
     3. Heading mirror — do RECONSTRUCTION headings echo gold-list titles too closely?
     4. 1:1 mapping suspect — does every gold target receive a neat reconstruction item, in order?
     5. Verdict: PASS / FLAG / FAIL with one-paragraph rationale.
   You read GOLD_WHYS to do this audit. The reconstruction was written WITHOUT seeing GOLD_WHYS, so any gold-only structure or vocabulary in the reconstruction is suspect.

D. runs/wave_NNN/scores/run_MMM.json
   Strict-schema, NUMBERS AND BOUNDED METADATA ONLY (per phase_two/SCORES_SCHEMA.md).
   - run_number: MMM as integer
   - timestamp: real ISO 8601 UTC from `date -u +%Y-%m-%dT%H:%M:%SZ` (NEVER a placeholder)
   - candidate.*: copy verbatim from runs/wave_NNN/plans/MMM/CANDIDATE_METADATA.json
   - evaluator.*: split into two halves if your schema supports; otherwise fill with scorer-side runtime
   - scores.*, denominators.*, low_confidence: per RUBRIC.md procedure

CRITICAL RULES:
- You may NOT modify runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md. It is frozen. If during scoring you notice missing rationale, that stays missing — you do not backfill.
- You may NOT read runs/wave_NNN/plans/<other slots>/ or runs/wave_NNN/reconstructions/<other slots>/ or runs/wave_NNN/scoring/<other slots>/.
- You may NOT read prd/. The PRD is encoded only via the gold list and rubric.
- Honest non-recovery: if RECONSTRUCTION says NOT RECOVERABLE, that scores as none recovery for that why; do NOT mentally fill it in.

READ ALLOWLIST:
- phase_two/  (entire)
- runs/wave_NNN/plans/MMM/PLAN.md and CANDIDATE_METADATA.json
- runs/wave_NNN/reconstructions/MMM/RECONSTRUCTION.md (read-only)
- runs/wave_NNN/scoring/MMM/  (your slot)
- AGENTS.md, CLAUDE.md, GEMINI.md, README.md  (orientation)

FORBIDDEN to read:
- prd/
- runs/wave_NNN/plans/<other slots>/
- runs/wave_NNN/reconstructions/<other slots>/
- runs/wave_NNN/scoring/<other slots>/
- runs/wave_K/ for K != NNN

WRITE ALLOWLIST: exactly four files —
- runs/wave_NNN/scoring/MMM/REPORT.md
- runs/wave_NNN/scoring/MMM/REPORT.html
- runs/wave_NNN/scoring/MMM/VALIDITY_AUDIT.md
- runs/wave_NNN/scores/run_MMM.json

When done, reply with a brief summary (under 250 words):
- The five score numbers (planning, fidelity, combined, system-level, feature-level)
- Validity audit verdict
- Key finding (1-2 sentences)
- Operational issues encountered
- Confirmation all four files written and you stayed within allowlists
```

Specify `model: <scorer_model>` per harness API.

After scorers return: orchestrator-side audit on writes (`find ... -newer ... -type f`), append warnings to METADATA.json.

---

## Wave completion

When phase 2B finishes for the wave:

1. Update `runs/wave_NNN/METADATA.json` with `completed_at: "<real ISO 8601 UTC>"`.
2. Compute wave-level summary: count of plans, reconstructions, scorings, average planning quality, average intent fidelity, count + breakdown of audit warnings.
3. Tell the user:
   ```
   Wave NNN complete. <K> runs in runs/wave_NNN/.
   Average planning: <X>%. Average fidelity: <Y>%.
   Validity audit verdicts: PASS=<a>, FLAG=<b>, FAIL=<c>.
   Per-run reports: runs/wave_NNN/scoring/MMM/REPORT.html.
   Audit warnings: <list, or "none">.
   ```

---

## What you do NOT do

- Do not auto-archive other waves.
- Do not touch other waves' folders.
- Do not run any single subagent in two phase roles. A reconstructor never scores; a scorer never plans.
- Do not silently overwrite an existing wave (atomic mkdir prevents this).
- Do not let a 2A reconstructor see anything but its assigned PLAN. This is the validity contract.
- Do not let a 2B scorer modify a frozen RECONSTRUCTION. This is the freeze rule.

---

## Constraints across all phases

- **Subagents stay in scope.** Their prompts spell out exact read and write allowlists.
- **Strict schema for SCORES.json** — numbers + bounded metadata only, per `phase_two/SCORES_SCHEMA.md`.
- **Real timestamps everywhere.** ISO 8601 UTC from the shell — never a placeholder.
- **Honest non-recovery.** Reconstructors mark `NOT RECOVERABLE FROM PLAN` rather than confabulate.
- **`.gitignore` is repo hygiene only, not isolation.** All scoping is enforced by prompt + audit.

---

## Optional harness-specific hardening

- **Cursor CLI:** `.cursor/agents/planner.md`, `.cursor/agents/reconstructor.md`, `.cursor/agents/scorer.md` ship as subagent definitions.
- **Codex CLI:** `--sandbox workspace-write --workspace-root <path>` per spawn.
- **Claude Code:** prompt-based scoping + post-hoc audit is the contract.

These configs are belt-and-suspenders on top of prompt allowlists. Their absence does NOT loosen the rules.
