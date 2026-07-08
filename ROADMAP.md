# Roadmap

This is the working document for the project. It records what is done, what is next,
and how to resume work — independent of any single conversation or session.

**Rule of engagement:** every milestone is executed as *issue → branch → PR → review → merge*.
Update this file in the same PR whenever a milestone's status changes.

## Project goal

Build a benchmark that makes real estate proposal quality measurable, then let an agent
improve itself against it using a Darwin Gödel Machine (DGM) loop.
Design: [docs/agent_design.md](docs/agent_design.md) · Benchmark: [benchmark_design.md](benchmark_design.md) · Rubric: [scoring_rubric.md](scoring_rubric.md)

## Status board

| # | Milestone | Status | Issue / PR |
|---|---|---|---|
| M0 | Benchmark design + scoring rubric | ✅ Done | initial commits |
| M0.5 | Agent + DGM loop design | ✅ Done | #1, PR #2 (merged) |
| M1 | **Golden set v1** (dataset) | 🔜 Next | — |
| M2 | Scaffold Phase 1 — single agent run | ⬜ Planned | — |
| M3 | Scoring pipeline (benchmark executable) | ⬜ Planned | — |
| M4 | DGM evolution loop | ⬜ Planned | — |
| M5 | Baseline run + build-log articles | ⬜ Planned | — |
| M6 | Real-data golden set (expert labels) | ⬜ Later | — |

## Milestones

### M1 — Golden set v1 (dataset)

The dataset is the heart of the system; nothing downstream can run without it.

Deliverables (all files under `golden/`):

- [ ] `taxonomy.yaml` — fixed objective taxonomy (6 classes from benchmark_design.md) + the 4 strategy axes and their legal values
- [ ] `articles/pool_*.yaml` — synthetic property pools, 20–30 properties each, numerically self-consistent (price vs rent vs yield), with deliberate distractors (high headline yield + vacancy risk, etc.)
- [ ] `cases/case_*.yaml` — 10–15 cases in `input` / `golden` two-section format, including ≥2 `no_match` cases; port the 5 cases from `examples/synthetic_cases.jsonl` (fixing the label-leak: objectives must be inferable from raw signals, never stated in `input`)
- [ ] `template/golden_set_template.xlsx` + `xlsx_to_yaml.py` — the intake channel for future expert-labeled real data
- [ ] 2 holdout cases kept out of the training list (marked `holdout: true`)

Acceptance: every case validates against the schema; a human review confirms each golden
answer is defensible sales judgment; `examples/synthetic_cases.jsonl` is retired.

**Human tasks:** review golden labels for sales-judgment sanity (only step that needs a person).

### M2 — Scaffold Phase 1: single agent run

- [ ] `core/schema.py`, `core/llm/` (Anthropic + OpenAI-compat + registry), `core/env/` (mock API + tools)
- [ ] Seed `agent/` (tool-use loop over the 3 layers, structured JSON output)
- [ ] `run_agent.py`, `config/config.yaml`

Acceptance: one case runs end-to-end against a real LLM; the tool-call trace shows the agent
querying customer + property pool through the mock API.

**Human tasks:** provide `ANTHROPIC_API_KEY` (or OpenRouter key) in the environment.

### M3 — Scoring pipeline

- [ ] `evaluation/`: scenario loader, metrics (objective match / coverage / Hit@3 / MRR / fact-check / no-match), LLM judge, scorer with gate caps, human-sheet export, integrity hashing
- [ ] Subprocess-isolated runner

Acceptance: the seed agent gets a stable score on the full set; gate-trigger tests pass
(fabricated number → cap; forced proposal on a no-match case → cap); judge repeatability
is acceptable across repeated scoring of the same output.

### M4 — DGM evolution loop

- [ ] `dgm/`: controller, archive, parent selector (paper formula), self-modify (diff mutation proof), validator (3 zero-cost gates)
- [ ] `run_dgm.py`

Acceptance: `--generations 3` completes with a lineage in the archive; a deliberately broken
mutation lands as invalid without crashing the loop; editing one byte of `evaluation/` aborts
the run (integrity check).

### M5 — Baseline + build log

- [ ] Run the seed agent as the baseline, record per-layer failures
- [ ] Run a short evolution, compare generations on the holdout cases
- [ ] Write articles 1–4 from the README build-log list using real results

### M6 — Real-data golden set (later, off-repo work)

- [ ] Extract 30–100 real cases (stratified), anonymize
- [ ] Expert labels for purpose / strategy / best proposal per case
- [ ] Import through the xlsx channel; keep synthetic set as a regression suite

**Human tasks:** the extraction, anonymization and expert labeling happen entirely outside
this public repo; only anonymized, reviewed output may enter it (see Safety And Data Boundary).

## How to resume with an AI agent

Point the agent at this file. Suggested kickoff per milestone:

> Read ROADMAP.md, docs/agent_design.md, benchmark_design.md and scoring_rubric.md in
> tianxiao1430-jpg/real-estate-agent-benchmark. Execute milestone M<N>: open an issue,
> work on a branch, open a PR that also updates the ROADMAP status board.
> Follow the frozen-zone/genome rules and the case schema from docs/agent_design.md.

Constraints that always apply:

- Synthetic data only in this repo; no real client/property/internal data.
- `input` sections must never contain golden labels.
- Scoring code (`evaluation/`) and golden data may not be touched by anything the agent
  self-modifies; keep the frozen-zone boundary from docs/agent_design.md.

## Decision log

| Date | Decision |
|---|---|
| 2026-07 | Benchmark evaluates the 3-layer reasoning chain (purpose → strategy → proposal), scored per layer + end-to-end |
| 2026-07 | Human-rubric dimensions are proxied by a fixed LLM judge inside the loop; formal evaluation stays human (exported sheets) |
| 2026-07 | Start with synthetic golden set; xlsx→YAML channel prepared for expert-labeled real data |
| 2026-07 | Agent works through mock workflow tools (customer lookup → search → detail → yield), not prompt-injected data |
| 2026-07 | Multi-provider LLM layer (Anthropic native + OpenAI-compatible); agent / self_modify / judge configured independently |
| 2026-07 | Develop and validate the full loop in this public repo on synthetic data first; any internal adaptation happens elsewhere by swapping the data source and mock API fields |
