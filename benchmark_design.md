# Benchmark Design

## Objective

Design a small, domain-specific benchmark for evaluating real estate proposal agents.

The benchmark should answer one question:

> Did the agent recommend the right kind of property for this investor, not just write a persuasive proposal?

## Core Assumption

The hardest failure is often upstream. If the agent misunderstands the investor's purpose, the final proposal can look polished while still being wrong.

For that reason, this benchmark separates the task into layers instead of using one vague score for "proposal quality".

## Evaluation Layers

### 1. Purpose Alignment

The agent receives a synthetic investor profile and must identify the primary investment objective.

Possible objectives:

- Stable monthly cash flow.
- Long-term asset appreciation.
- Lower operational risk.
- Tax or inheritance planning.
- Portfolio diversification.
- No suitable proposal yet.

The benchmark should penalize confident but wrong objective detection.

### 2. Strategy Formulation

The agent must translate the investor objective into search criteria.

Possible criteria:

- Property type.
- Location preference.
- Budget range.
- Target yield range.
- Vacancy risk.
- Repair and renovation risk.
- Loan burden.
- Management complexity.
- Exit liquidity.

The benchmark should not reward high headline yield if the investor profile prioritizes stability.

### 3. Proposal Fit

The agent must choose from synthetic property candidates and explain the recommendation.

The final proposal should:

- Match the investor's purpose.
- Use the strategy it claimed to use.
- State tradeoffs honestly.
- Avoid hallucinated numbers.
- Reject unsuitable options when needed.

## Scoring Model

The first version uses three kinds of checks:

1. Automatic metrics
   - Exact or structured matching for objective labels, risk flags, and selected property IDs.

2. LLM judge
   - Fixed model and fixed prompt.
   - Scores qualitative dimensions such as reasoning consistency and tradeoff explanation.

3. Cutoff gates
   - Hard penalties for critical failures.
   - Examples: fabricated financial numbers, ignoring required constraints, recommending a property when all options should be rejected.

## Safety And Data Boundary

This benchmark starts with synthetic data only.

It must not contain:

- Real client names.
- Real addresses.
- Real transaction records.
- Internal company documents.
- Personally identifiable information.

Any future use of real data requires separate review and anonymization.

## First Milestone

Create 3 to 5 synthetic investor cases and 6 to 10 synthetic property candidates.

Then run a baseline prompt-based agent and record:

- Which layer failed.
- Whether the failure was obvious from the final proposal.
- Whether the benchmark caught the failure.
