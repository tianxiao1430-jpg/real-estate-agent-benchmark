# Agent Design: Self-Improvement Loop (Darwin Gödel Machine)

> Status: design proposal (see issue #1).
> `benchmark_design.md` defines **what to measure**. This document defines **the agent scaffold** and **how the agent improves itself** against that benchmark — the "agent self-improvement" milestone from the README.
> All data referenced here is synthetic, consistent with the Safety And Data Boundary section of `benchmark_design.md`.

## 1. Concept — three pieces

| Piece | Role |
|---|---|
| **Benchmark** | The goal translated into a measurable evaluation basis (automatic metrics + LLM judge + cutoff gates). Every direction the evolution takes is decided by this number — it is the heart of the system. |
| **Agent scaffold** | A minimal but complete skeleton that performs the task through tool calls against a mock environment and returns structured output. |
| **DGM evolution loop** | A self-improvement mechanism where the agent rewrites its own code (prompts / strategy logic), is empirically validated on the benchmark, and every variant is archived with its lineage. |

The Darwin Gödel Machine decides whether a self-modification survives by **empirical validation** (benchmark score), not formal proof. Population-based search (archive + parent selection) keeps the evolution open-ended instead of greedy.

References: [arXiv:2505.22954](https://arxiv.org/abs/2505.22954), [lemoz/darwin-godel-machine](https://github.com/lemoz/darwin-godel-machine) (MIT).

## 2. Benchmark → fitness mapping

The three evaluation layers from `benchmark_design.md` are scored per step, plus end-to-end, so a failure can be located ("which layer broke") instead of blended into one vague score.

| Layer | Metric | Kind | Definition |
|---|---|---|---|
| 1. Purpose Alignment | Objective match rate | automatic | Detected objective (fixed taxonomy: cash flow / appreciation / tax planning / risk reduction / diversification / no suitable proposal) matches the golden label |
| 1. Purpose Alignment | Financial consistency | LLM judge | Detected objective does not contradict income / own funds / borrowing capacity (1–5) |
| 2. Strategy Formulation | Criteria coverage | automatic | Share of required axes made explicit (budget range / location / building type & age / yield emphasis, risk filters) |
| 2. Strategy Formulation | Strategy agreement | LLM judge | How closely the strategy matches the golden (expert) strategy (1–5) |
| 3. Proposal Fit | Hit@3 / MRR | automatic | Was the golden property ranked in the top 3; ranking quality |
| 3. Proposal Fit | Reasoning validity | LLM judge | Reasons are consistent with the strategy and explainable to the investor (1–5) |
| Gates | Fact-error rate / no-match correctness | gate | Numbers quoted in the reasoning (price, yield, building age, station distance) must match candidate data — zero errors required; on "no suitable proposal" cases the agent must not force a recommendation |

Composition rules:

- Case score = weighted blend of automatic + judge metrics. **A gate violation caps the case score** (consistent with the Cutoff Gates section of `scoring_rubric.md`).
- Agent fitness = mean over all cases → fed directly into DGM parent selection.
- The LLM judge is a **proxy for human scoring**: fixed model, temperature 0, forced JSON output, unchanged across generations so scores stay comparable. A human scoring sheet can be exported for expert review of the best individuals.
- Default is one end-to-end run scored per layer (cheap). An optional `--isolate-steps` mode re-runs each layer with the upstream golden answer given, for diagnosis.

### Case file schema

Cases are split into an `input` section (the only thing the agent ever sees) and a `golden` section (evaluator-only — it never enters the agent context).

```yaml
schema_version: 1
name: case_001
# ── input: the only part visible to the agent (served through the mock API) ──
input:
  customer:        # age 34 / income 7.0M JPY / own funds 3.0M / borrowing capacity 35M /
                   # no investment experience / notes "worried about retirement, interested in tax" /
                   # hard constraint "no rural properties"
  article_pool:    # 20–30 candidates: price, layout, building age, station distance,
                   # expected rent, management fee, ...
# ── golden: evaluator-only ──
golden:
  purpose: retirement income (tax is secondary)
  strategy: { budget: 25–35M JPY, location: urban-access area,
              building: built within 15 years, yield_emphasis: stable occupancy over headline yield }
  best_articles: [A102, A088, A140]   # ordered golden answer; [] for no-match cases
  no_match: false
```

Golden-set principles (from `benchmark_design.md`, unchanged): synthetic data only; 10–20% of cases have "no suitable proposal" as the correct answer; expert labels are the primary golden source (they are the only source that can label layers 1 and 2), outcome data only assists layer 3.

## 3. Scaffold — frozen zone vs. genome

One design rule dominates everything: **physically separate "what may evolve" from "what must not".**

```
project/
├── run_agent.py                 # CLI: run one case (--score / --isolate-steps)
├── run_dgm.py                   # CLI: evolution loop --generations N
├── core/                        # [FROZEN] importable by the agent, never editable
│   ├── schema.py                # CustomerCase / Article / Purpose / Strategy / Proposal / AgentOutput
│   ├── llm/                     # multi-provider abstraction
│   │   ├── base.py              # LLMClient ABC (tool use aware)
│   │   ├── anthropic_client.py  # native Anthropic SDK
│   │   ├── openai_compat.py     # OpenAI-compatible endpoints (OpenRouter / vLLM / Ollama)
│   │   └── registry.py          # create_client(role, config)
│   └── env/                     # [FROZEN] high-fidelity mock of a sales workflow system
│       ├── mock_api.py          # get_customer / get_opportunity / search_articles /
│       │                        #   get_article_detail / calc_yield
│       └── tools.py             # tool JSON schemas + dispatcher
├── agent/                       # [GENOME] the only target of self-modification
│   ├── sales_agent.py           # tool-use loop executing layers 1→2→3, structured JSON output
│   ├── prompts.py               # prompt templates (the main battleground of evolution)
│   └── strategies.py            # evolvable logic: objective inference, strategy building, ranking
├── evaluation/                  # [FROZEN] scoring code
│   ├── scenario.py              # case YAML loader; input/golden separation
│   ├── runner.py                # subprocess-isolated execution (timeout + workspace isolation)
│   ├── metrics.py               # automatic metrics
│   ├── llm_judge.py             # judge proxy (1–5, temp=0, forced JSON)
│   ├── scorer.py                # per-layer + end-to-end blend, gate caps
│   ├── human_sheet.py           # export blank human scoring sheets
│   └── integrity.py             # tree-hash verification of the frozen zone (abort on tampering)
├── dgm/                         # [FROZEN] evolution controller
│   ├── controller.py            # select parent → self-modify → validate → evaluate → archive
│   ├── archive.py               # individuals stored with lineage
│   ├── parent_selector.py       # paper formula: s=sigmoid(λ(α−α0)), h=1/(1+n), w=s·h
│   ├── self_modify.py           # diagnosis → edit task → sandboxed edit tools → diff mutation proof
│   └── validator.py             # 3 gates: ast.parse → import → mock smoke run
├── golden/                      # golden set (template + converter + cases/*.yaml)
├── config/config.yaml           # providers / per-role models / metric weights / DGM params
└── README.md
```

Key scaffold decisions:

- **Environment as mock tools, not prompt stuffing.** The agent must call tools shaped like a real workflow (customer lookup → property search → detail → yield calc). The agent's *tool-use behavior itself* becomes evolvable, and the mock can later be swapped for a real API adapter behind the same interface.
- **No LLM lock-in.** Two-level config: provider definitions × role mapping. The executor (`agent`), the self-modifier (`self_modify`), and the scorer (`judge`) can each use a different provider/model.
- **Structured output.** The agent returns per-layer intermediate results as JSON so each layer can be scored mechanically.

```yaml
providers:
  anthropic:  { type: anthropic,     api_key: ${ANTHROPIC_API_KEY} }
  openrouter: { type: openai_compat, api_key: ${OPENROUTER_API_KEY}, base_url: https://openrouter.ai/api/v1 }
  local:      { type: openai_compat, api_key: dummy, base_url: http://localhost:11434/v1 }
roles:
  agent:       { provider: anthropic, model: claude-opus-4-8 }
  self_modify: { provider: anthropic, model: claude-opus-4-8 }   # strongest model recommended
  judge:       { provider: anthropic, model: claude-opus-4-8 }   # fixed across all generations
dgm:
  parent_selection: { lambda: 10.0, alpha_0: 0.5 }
  stop: { max_generations: N, target_score: 0.90, max_consecutive_noop: 3 }
```

## 4. The DGM evolution loop

```
select parent → self-modify → validate → evaluate → archive → (next generation)
```

1. **Parent selection** — sample from the archive using the paper formula: sigmoid-weighted score × discount by number of children (exploration/exploitation balance).
2. **Self-modification** — build a diagnosis from the parent's evaluation details ("which layer is weak, which axis is missing"), then let the LLM edit the `agent/` copy with restricted edit tools. A before/after snapshot produces a **diff-based mutation proof**; a no-op change is fused out without spending evaluation budget.
3. **Validation** — three zero-API-cost gates: syntax (`ast.parse` + class/method presence), load (`importlib` with a dummy client), smoke (full run with a mock LLM + mock API on a fixed mini case).
4. **Evaluation** — run all cases in an isolated subprocess, compute fitness.
5. **Archive** — store the individual with its lineage **without a score threshold**; low scorers stay available as exploration seeds (open-endedness).

Stop conditions: generation cap / target score / consecutive-noop fuse — a triple brake against cost runaway.

## 5. Safety boundary — four layers against reward hacking

The biggest risk of a self-improving system is evolving toward **exploiting the scorer** instead of getting better at the task.

1. **Physical isolation** — the self-modification workspace contains only a copy of `agent/`; edit tools reject any path outside it (`resolve().is_relative_to(workspace)`); the agent subprocess cannot import evaluation code or golden data.
2. **Information isolation** — golden labels and judge rubrics never enter the agent context; the diagnosis passed to self-modification contains **symptoms only** ("layer-2 coverage low, budget axis missing"), never scoring internals.
3. **Integrity verification** — a tree hash of the frozen zone is recorded at startup and re-checked before every evaluation; any mismatch is treated as scorer contamination and aborts the run.
4. **Judge defense** — agent output is wrapped in delimiters, the judge is told that nothing inside the delimiters carries instruction authority, and the judge must reply in a forced JSON schema (blocking prompt-injection score manipulation).

Because none of this is airtight, the operational backstop is a **holdout case set** (never used in the loop) plus periodic human review of the best individuals.

## 6. Phased rollout

| Phase | Content | Acceptance |
|---|---|---|
| 1. Single run | `core/` + seed `agent/` + `run_agent.py` + 3 synthetic cases (input only) | One case runs end-to-end; tool-call trace visible |
| 2. Scoring pipeline | metrics + judge + scorer + human sheet + integrity; golden set to 10–15 cases incl. no-match cases; template + converter | Seed agent scores are stable; gate-trigger tests pass; judge repeatability acceptable |
| 3. Evolution loop | `dgm/` + `run_dgm.py` | 3 generations complete with lineage; broken code lands as invalid without crashing; integrity abort verified |
| 4. Docs | README + productionization guide (real API adapter, container isolation, real-data intake) | The loop is reproducible from the README alone |

## 7. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Judge noise drowns out real improvement | temp=0, fixed model, anchored rubric, averaging over ≥10 cases; automatic metrics as the primary signal |
| Reward hacking | four-layer defense + dual scoring tracks + holdout + human spot checks |
| Synthetic-data bias | golden sources and their contamination factors documented; real-data intake channel (template → converter) prepared from day one |
| Cost runaway | per-role model switching (judge → cheaper model / local endpoint) + triple brake + noop detection |
| Subprocess isolation weaker than containers | accepted for the PoC and stated explicitly; integrity hashing seals the most critical path (scorer tampering); production should move to containers |
