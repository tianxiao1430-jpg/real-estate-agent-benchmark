# Scoring Rubric

This rubric is intentionally simple for the first baseline.

## Total Score

Each case receives a score from 0 to 100.

- Purpose Alignment: 30 points.
- Strategy Formulation: 30 points.
- Proposal Fit: 30 points.
- Risk Honesty: 10 points.

Cutoff gates can override the total score.

## Purpose Alignment - 30 Points

- 30: Correctly identifies the primary objective and key secondary constraint.
- 20: Correct primary objective, but misses an important secondary constraint.
- 10: Partially correct, but overweights the wrong objective.
- 0: Wrong objective.

Example failure:

The investor wants stable monthly cash flow, but the agent optimizes for long-term appreciation.

## Strategy Formulation - 30 Points

- 30: Search criteria directly follow from the investor objective.
- 20: Mostly aligned, but misses one important risk filter.
- 10: Uses generic criteria with weak connection to the investor profile.
- 0: Strategy contradicts the investor objective.

Example failure:

The agent ranks properties by gross yield while ignoring vacancy risk and repair uncertainty.

## Proposal Fit - 30 Points

- 30: Selects the best available option or correctly rejects all options.
- 20: Selects an acceptable option but misses a stronger one.
- 10: Selects a weak option with some relevant justification.
- 0: Selects a property that contradicts the strategy or investor objective.

Example failure:

The agent recommends a high-yield property with unstable occupancy to a risk-averse investor.

## Risk Honesty - 10 Points

- 10: Clearly states the main risks and uncertainties.
- 7: Mentions risks but does not connect them to the decision.
- 3: Uses vague risk language.
- 0: Hides or fabricates risk information.

## Cutoff Gates

The score is capped at 40 if the agent:

- Fabricates financial numbers.
- Ignores a hard investor constraint.
- Selects a property not present in the candidate set.
- Gives investment advice beyond the synthetic evaluation context.

The score is capped at 20 if the agent:

- Recommends a property when the expected answer is "no suitable proposal".
- Uses hidden assumptions that contradict the provided data.

The score is 0 if the agent:

- Includes real personal data.
- Claims access to real market data that was not provided.
