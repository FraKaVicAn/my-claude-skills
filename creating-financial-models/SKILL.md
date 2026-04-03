---
name: creating-financial-models
description: This skill provides an advanced financial modeling suite with DCF analysis, sensitivity testing, Monte Carlo simulations, and scenario planning for investment decisions
origin: anthropics/claude-cookbooks/skills/custom_skills/creating-financial-models
tools: Read, Write, Edit, Bash
---

# Financial Modeling Suite

A comprehensive financial modeling toolkit for investment analysis, valuation, and risk assessment using industry-standard methodologies.

## Core Capabilities

### 1. Discounted Cash Flow (DCF) Analysis
- Build complete DCF models with multiple growth scenarios
- Calculate terminal values using perpetuity growth and exit multiple methods
- Determine weighted average cost of capital (WACC)
- Generate enterprise and equity valuations

### 2. Sensitivity Analysis
- Test key assumptions impact on valuation
- Create data tables for multiple variables
- Generate tornado charts for sensitivity ranking
- Identify critical value drivers

### 3. Monte Carlo Simulation
- Run thousands of scenarios with probability distributions
- Model uncertainty in key inputs
- Generate confidence intervals for valuations
- Calculate probability of achieving targets

### 4. Scenario Planning
- Build best/base/worst case scenarios
- Model different economic environments
- Test strategic alternatives
- Compare outcome probabilities

## Input Requirements

### For DCF Analysis
- Historical financial statements (3-5 years)
- Revenue growth assumptions
- Operating margin projections
- Capital expenditure forecasts
- Working capital requirements
- Terminal growth rate or exit multiple
- Discount rate components (risk-free rate, beta, market premium)

### For Sensitivity Analysis
- Base case model
- Variable ranges to test
- Key metrics to track

### For Monte Carlo Simulation
- Probability distributions for uncertain variables
- Correlation assumptions between variables
- Number of iterations (typically 1,000–10,000)

### For Scenario Planning
- Scenario definitions and assumptions
- Probability weights for scenarios
- Key performance indicators to track

## Output Formats

### DCF Model Output
- Complete financial projections
- Free cash flow calculations
- Terminal value computation
- Enterprise and equity value summary
- Valuation multiples implied
- Excel workbook with full model

### Sensitivity Analysis Output
- Sensitivity tables showing value ranges
- Tornado chart of key drivers
- Break-even analysis
- Charts showing relationships

### Monte Carlo Output
- Probability distribution of valuations
- Confidence intervals (e.g., 90%, 95%)
- Statistical summary (mean, median, std dev)
- Risk metrics (VaR, probability of loss)

### Scenario Planning Output
- Scenario comparison table
- Probability-weighted expected values
- Decision tree visualization
- Risk-return profiles

## Model Types Supported

1. **Corporate Valuation** — Mature companies, growth stage, turnarounds
2. **Project Finance** — Infrastructure, real estate, energy
3. **M&A Analysis** — Acquisition valuations, synergy modeling, accretion/dilution
4. **LBO Models** — Leveraged buyout, IRR/MOIC returns, debt capacity

## DCF Quick Reference

```python
# WACC calculation
def calculate_wacc(equity_value, debt_value, cost_equity, cost_debt, tax_rate):
    total = equity_value + debt_value
    wacc = (equity_value/total * cost_equity) + (debt_value/total * cost_debt * (1 - tax_rate))
    return wacc

# Terminal value (Gordon Growth Model)
def terminal_value_ggm(fcf_final, growth_rate, wacc):
    return fcf_final * (1 + growth_rate) / (wacc - growth_rate)

# Terminal value (Exit Multiple)
def terminal_value_exit(ebitda_final, exit_multiple):
    return ebitda_final * exit_multiple

# Enterprise value from DCF
def enterprise_value(free_cash_flows, terminal_value, wacc):
    import numpy as np
    periods = len(free_cash_flows) + 1
    discount_factors = [(1 + wacc)**(-t) for t in range(1, periods + 1)]
    pv_fcfs = sum(fcf * df for fcf, df in zip(free_cash_flows, discount_factors[:-1]))
    pv_tv = terminal_value * discount_factors[-1]
    return pv_fcfs + pv_tv
```

## Monte Carlo Example

```python
import numpy as np

def monte_carlo_dcf(n_simulations=10_000):
    results = []
    for _ in range(n_simulations):
        # Sample from distributions
        revenue_growth = np.random.normal(0.08, 0.03)   # 8% ± 3%
        ebitda_margin  = np.random.normal(0.20, 0.02)   # 20% ± 2%
        wacc           = np.random.normal(0.10, 0.01)   # 10% ± 1%
        terminal_growth= np.random.normal(0.025, 0.005) # 2.5% ± 0.5%
        # ... build DCF and calculate EV
        results.append(enterprise_value_from_inputs(revenue_growth, ebitda_margin, wacc, terminal_growth))

    results = np.array(results)
    return {
        "mean": results.mean(), "median": np.median(results),
        "p10": np.percentile(results, 10), "p90": np.percentile(results, 90),
        "prob_above_target": (results > target_value).mean()
    }
```

## Best Practices

- Use multiple valuation methods (DCF + comps + precedent transactions)
- Apply appropriate risk adjustments for stage and market
- Document every assumption explicitly
- Stress test extreme cases, not just base ± 10%
- Consider correlation between inputs in Monte Carlo
- Models are only as good as their assumptions — clearly state limitations

## Quality Checks

1. Balance sheet balancing checks
2. Cash flow reconciliation
3. Circular reference resolution (WACC ↔ equity value)
4. Sensitivity bound validation
5. Statistical validation of Monte Carlo results

## Example Prompts

- "Build a DCF model for this technology company using the attached financials"
- "Run a Monte Carlo simulation on this acquisition model with 5,000 iterations"
- "Create sensitivity analysis showing impact of growth rate and WACC on valuation"
- "Develop three scenarios for this expansion project with probability weights"

## Links

- [Cookbook: skills/custom_skills/creating-financial-models](https://github.com/anthropics/claude-cookbooks/tree/main/skills/custom_skills/creating-financial-models)
