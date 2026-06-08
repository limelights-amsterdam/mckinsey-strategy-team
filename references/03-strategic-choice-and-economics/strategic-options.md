---
name: strategic-options
description: Generates and compares strategic options with clear decision criteria, trade-offs, and recommendation logic. Use for strategy options, path selection, growth choices, market entry choices, build-buy-partner, or "what are our options?" questions.
---

# Strategic Options

## When To Use

Use this skill when the user needs alternatives before committing to a strategy. It helps avoid false binaries and creates a defensible path from options to recommendation.

## McKinsey-Style Approach

Develop a small set of distinct, viable options. Compare them against decision criteria that matter. Recommend a path with trade-offs visible.

## Workflow

1. Clarify the strategic decision and constraints.
2. Generate mutually distinct options.
3. Define decision criteria.
4. Assess each option against attractiveness, feasibility, risk, economics, and strategic fit.
5. Identify hybrids or sequencing if useful.
6. Recommend the best path and what must be true.

## Output Format

```markdown
# Strategic Options

## Decision
[Decision being made.]

## Options
| Option | Description | Upside | Trade-Off | What Must Be True |
|---|---|---|---|---|

## Evaluation
| Criteria | Option A | Option B | Option C |
|---|---|---|---|

## Recommendation
[Preferred option and rationale.]

## Next Tests
[Evidence needed before committing.]
```

## Quality Bar

- Options must be meaningfully different.
- Criteria must reflect the user's decision, not generic attractiveness.
- Include trade-offs and risks.
- State what evidence would change the recommendation.
