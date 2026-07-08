# Real Estate Agent Benchmark

This repository is a public build log for a simple question:

> Before trying to build a smarter real estate proposal agent, can I first define what a good proposal means for my own business context?

Most AI benchmarks compare models in general tasks. This project is narrower. It is about turning real estate proposal judgment into measurable evaluation steps.

## Why This Exists

Real estate proposal agents can sound convincing while still optimizing for the wrong goal. A proposal may be well written, but wrong for the investor.

For example, a client seeking stable monthly cash flow should not be evaluated the same way as a client seeking long-term asset appreciation. A property with a high headline yield may still be a poor fit if vacancy risk, repair uncertainty, loan burden, or tenant stability is ignored.

The first goal of this repository is not to build an autonomous agent. The first goal is to define a benchmark that makes those failures visible.

## What The Benchmark Measures

The first version uses three evaluation layers:

1. Purpose Alignment
   - Did the agent identify the investor's real objective?
   - Examples: cash flow, asset appreciation, tax planning, risk reduction.

2. Strategy Formulation
   - Did the agent translate that objective into the right search strategy?
   - Examples: location, price range, yield range, vacancy risk, renovation risk, loan burden.

3. Proposal Fit
   - Did the final recommendation match the strategy and explain the tradeoffs honestly?
   - Examples: why this property fits, what risks remain, when the answer should be "no suitable proposal".

## What Is In Scope

- Synthetic investor profiles.
- Synthetic property candidates.
- A transparent scoring rubric.
- Baseline runs against simple prompt-based agents.
- Later experiments with agent self-improvement against the benchmark.

## What Is Out Of Scope For Now

- Real client data.
- Real property transaction data.
- Investment advice.
- Production deployment.
- Autonomous purchase or sales decisions.

All examples in this repository are synthetic and are only for evaluation design.

## Current Status

Phase 0: benchmark design.

The next milestone is to create a small synthetic dataset and run a baseline agent to see where it fails.

## Working Document

- [ROADMAP.md](ROADMAP.md) — the living work plan: status board, milestone checklists, acceptance criteria, and how to resume work with an AI agent.

## Design Documents

- [docs/agent_design.md](docs/agent_design.md) — agent scaffold + Darwin Gödel Machine (DGM) self-improvement loop: how an agent will be evolved against this benchmark (fitness mapping, frozen-zone/genome split, evolution loop, anti-reward-hacking safety boundary). See issue #1.

## Build Log

- Article 0: Why I need a benchmark before I need a smarter real estate agent.
- Article 1: The first baseline failed in an interesting way.
- Article 2: Why "good proposal" is the wrong single metric.
- Article 3: What changed when the agent optimized against the benchmark.
- Article 4: Reward hacking and failure modes.
