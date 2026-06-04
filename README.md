# CARE — Capture And Recovery Eval

A benchmark for AI-written plans that you **check out and run yourself**. Point your own coding agent at the product spec in this repo; it produces an implementation plan; then a separate, isolated phase evaluates that plan — giving you a score you can compare across models, harnesses, and effort levels.

> ⚠️ **This repo is contamination-sensitive.** What CARE actually measures, and exactly how a plan is scored, is deliberately **not** written here — that detail is held out so a model being benchmarked can't read the README and write to the test. The full methodology lives on the results site.

📊 **Official results, leaderboard & methodology → [carebenchmark.metal-sole.com](https://carebenchmark.metal-sole.com/)**

## Watch — the story behind it

A walkthrough of how this benchmark was designed and built with AI agents:

<a href="https://youtu.be/mkH4N6IXnic">
  <img src="https://i.ytimg.com/vi/mkH4N6IXnic/maxresdefault.jpg" alt="How I Actually Used AI Agents to Build a Benchmark" width="560">
</a>

**[How I Actually Used AI Agents to Build a Benchmark](https://youtu.be/mkH4N6IXnic)** — Matt Maher

## Run it yourself

You drive CARE by talking to a coding agent that's open in this repo — Claude Code, Codex CLI, Gemini CLI, opencode, Cursor, or anything that reads the agent router files. There's no script to run: you ask in plain language and the agent routes itself to the right phase, reading only that phase's instructions.

```sh
git clone https://github.com/bladnman/care_benchmark.git
cd care_benchmark
```

Then, with the repo open in your agent:

1. **Generate plans (phase 1).** A candidate model reads the spec and writes plans. It never sees any grading material.

   > `Run 5 plans`
   > `Run 3 plans on gpt-5.5 at high effort`

   The agent claims a fresh `runs/wave_NNN/` and writes one plan per slot.

2. **Evaluate (phase 2).** In a clean agent context, ask it to evaluate what phase 1 produced. Evaluation runs walled off from planning.

   > `Evaluate the wave`

   The agent unpacks the sealed grading package, runs the evaluation, and reports averages plus per-run report paths.

3. **Read the results.** Open the generated `runs/wave_NNN/scoring/MMM/REPORT.html` (or the `REPORT.md` twin), or compare across models on the [results site](https://carebenchmark.metal-sole.com/).

To benchmark a different model or harness, run step 1 from a different agent — the candidate is simply whichever agent wrote the plan.

## Where the results live

The published numbers are rendered at **[carebenchmark.metal-sole.com](https://carebenchmark.metal-sole.com/)** — the easiest way to read and compare results. The raw data behind the site lives in this repo, on branches.

`main` is a **clean instance**: it ships the spec, the phase instructions, and the sealed grading package, but no run output (`runs/` is gitignored here). Every completed configuration is captured on its **own branch** under `results/…`, so the numbers live beside the exact instance that produced them.

```
git branch -a | grep '^  results/'      # list every captured configuration
git checkout results/codex-cli/cand-gpt-5.5__effort-extra-high/eval-gpt-5.5__effort-extra-high
open runs/wave_001/scoring/001/REPORT.html
```

A branch name decodes the full configuration:

```
results/<candidate-harness>[__<eval-harness>]/cand-<model>__effort-<x>/eval-<model>__effort-<x>
```

For example, `results/claude-code__codex-cli/cand-claude-4.8-opus__effort-max/eval-gpt-5.5__effort-extra-high` is **claude-4.8-opus** (run under Claude Code, *max* effort) evaluated by **gpt-5.5** (run under Codex CLI, *extra-high*). When the candidate and evaluator share a harness the prefix collapses to one segment, e.g. `results/codex-cli/…`. Models captured so far span Claude, GPT, Gemini, and a range of open models (DeepSeek, Qwen, Kimi, MiniMax, Grok, and others).

Each result branch carries the clean instance plus the full run tree under `runs/wave_NNN/`: the candidate's `PLAN.md`, the evaluation artifacts, machine-readable scores, and a human-readable `REPORT.html`/`REPORT.md` per run.

## Repo layout

| Location | Contents |
|---|---|
| [`prd/`](prd/) | The product spec a phase-1 candidate plans from (starts at `prd/1-START_HERE.md`) |
| [`PHASE_ONE_INSTRUCTIONS.md`](PHASE_ONE_INSTRUCTIONS.md) | Phase-1 workflow — the agent reads this, not you |
| [`phase_two.zip`](phase_two.zip) | Sealed grading package — unpacked only at evaluation, never during planning |
| `AGENTS.md` · `CLAUDE.md` · `GEMINI.md` | Tiny per-harness routers that send each agent to the right phase |
| `runs/` | Generated wave output — gitignored on `main`, captured on `results/…` branches |

## Contamination & integrity

CARE only means something if the model being benchmarked hasn't seen how it'll be graded. That's the whole reason for the structure here:

- **The methodology is held out on purpose.** What CARE measures and how plans are scored is documented on the [results site](https://carebenchmark.metal-sole.com/), not in this repo. The grading material itself stays sealed in `phase_two.zip` and is opened only inside an evaluation context that never touches planning.
- **If you're generating plans, don't go looking for the grading material** — let the phases stay separate. Phase separation is enforced by the phase instructions and a post-run validity audit, not by `.gitignore` (the ignore rules are just repo hygiene).
- **No harness-specific agent folders ship** (e.g. `.cursor/agents/`) — the phase instructions are runner-portable, so any agent can drive a wave.
