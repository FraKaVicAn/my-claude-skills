---
name: analyzing-financial-statements
description: This skill calculates key financial ratios and metrics from financial statement data for investment analysis — profitability, liquidity, leverage, efficiency, and valuation ratios
origin: anthropics/claude-cookbooks/skills/custom_skills/analyzing-financial-statements
tools: Read, Write, Edit, Bash
---

# Financial Ratio Calculator Skill

Comprehensive financial ratio analysis for evaluating company performance, profitability, liquidity, and valuation.

## Capabilities

Calculate and interpret:
- **Profitability**: ROE, ROA, Gross Margin, Operating Margin, Net Margin, EBITDA Margin
- **Liquidity**: Current Ratio, Quick Ratio, Cash Ratio
- **Leverage**: Debt-to-Equity, Interest Coverage, Debt Service Coverage, Net Debt/EBITDA
- **Efficiency**: Asset Turnover, Inventory Turnover, Receivables Turnover, Days Sales Outstanding
- **Valuation**: P/E, P/B, P/S, EV/EBITDA, EV/Revenue, PEG
- **Per-Share**: EPS, Book Value per Share, Dividend per Share, FCF per Share

## Input Formats

- CSV with financial line items
- JSON with structured financial statements
- Text description of key financial figures
- Excel files with financial statements

## Output Format

- Calculated ratios with values
- Industry benchmark comparisons (when available)
- Trend analysis (if multiple periods provided)
- Interpretation and insights
- Excel report with formatted results

## Ratio Reference

```python
# Profitability
gross_margin      = gross_profit / revenue
operating_margin  = operating_income / revenue
net_margin        = net_income / revenue
roe               = net_income / shareholders_equity
roa               = net_income / total_assets
roic              = nopat / invested_capital

# Liquidity
current_ratio     = current_assets / current_liabilities
quick_ratio       = (current_assets - inventory) / current_liabilities
cash_ratio        = cash_and_equivalents / current_liabilities

# Leverage
debt_to_equity    = total_debt / shareholders_equity
interest_coverage = ebit / interest_expense
net_debt_ebitda   = (total_debt - cash) / ebitda

# Efficiency
asset_turnover    = revenue / total_assets
inventory_turns   = cogs / average_inventory
dso               = (accounts_receivable / revenue) * 365

# Valuation
pe_ratio          = stock_price / eps
ev_ebitda         = enterprise_value / ebitda
pb_ratio          = stock_price / book_value_per_share
```

## Industry Benchmarks (General)

| Ratio | Weak | Average | Strong |
|-------|------|---------|--------|
| Current Ratio | < 1.0 | 1.0–2.0 | > 2.0 |
| Quick Ratio | < 0.5 | 0.5–1.0 | > 1.0 |
| Debt/Equity | > 2.0 | 0.5–2.0 | < 0.5 |
| Interest Coverage | < 2x | 2x–5x | > 5x |
| ROE | < 10% | 10–20% | > 20% |
| Net Margin | < 5% | 5–15% | > 15% |

*Note: Benchmarks vary significantly by industry — always compare to sector peers.*

## Multi-Period Trend Analysis

```python
def trend_analysis(ratios_by_year: dict) -> dict:
    """
    ratios_by_year: {2022: {ratio: value}, 2023: {...}, 2024: {...}}
    Returns trend direction and CAGR for each ratio.
    """
    trends = {}
    years = sorted(ratios_by_year.keys())
    for ratio in ratios_by_year[years[0]]:
        values = [ratios_by_year[y].get(ratio) for y in years]
        values = [v for v in values if v is not None]
        if len(values) >= 2:
            direction = "improving" if values[-1] > values[0] else "declining"
            n = len(values) - 1
            cagr = (values[-1] / values[0]) ** (1/n) - 1 if values[0] != 0 else None
            trends[ratio] = {"direction": direction, "cagr": cagr, "values": values}
    return trends
```

## Best Practices

1. Validate data completeness before calculations
2. Handle missing values appropriately
3. Consider industry context when interpreting ratios
4. Include period comparisons for trend analysis
5. Flag unusual or concerning ratios for deeper investigation
6. Use multiple ratios together — no single ratio tells the full story

## Example Prompts

- "Calculate key financial ratios for this company based on the attached financial statements"
- "What's the P/E ratio if the stock price is $50 and annual earnings are $2.50 per share?"
- "Analyze the liquidity position using the balance sheet data"
- "Compare this company's margins to industry benchmarks"
- "Show me a 3-year trend analysis of profitability ratios"

## Links

- [Cookbook: skills/custom_skills/analyzing-financial-statements](https://github.com/anthropics/claude-cookbooks/tree/main/skills/custom_skills/analyzing-financial-statements)
