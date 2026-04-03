---
name: factor-analysis
description: Build and evaluate alpha factors — signal construction, IC analysis, turnover, and performance attribution — using Alphalens or custom implementations
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Factor Analysis Skill

Research, validate, and decompose alpha factors: information coefficient (IC), quantile returns, turnover, and sector/market neutralization.

## When to Activate

- Evaluating a new alpha signal before integrating it into a portfolio
- Computing IC, ICIR, and quantile return spreads for a factor
- Detecting factor decay — how quickly does predictive power fade?
- Neutralizing a factor for market or sector bias
- Combining multiple factors into a composite signal

## Key Libraries

| Library | Use Case |
|---------|----------|
| `alphalens-reloaded` | Full factor tearsheet, IC, quantile, turnover |
| `pandas`, `scipy` | Custom IC and neutralization |
| `statsmodels` | Cross-sectional regression factor attribution |

## Data Format for Alphalens

Alphalens expects:
- `factor`: `pd.Series` with `MultiIndex (date, asset)`, values = raw factor scores
- `prices`: `pd.DataFrame` with `date` index and `asset` columns (forward-looking)

```python
import alphalens as al

# Build MultiIndex factor series
factor_data = factor_df.stack()              # (date, ticker) → factor value
factor_data.index.names = ["date", "asset"]

# Compute forward returns and align
factor_and_returns = al.utils.get_clean_factor_and_forward_returns(
    factor_data,
    prices,
    quantiles=5,
    periods=(1, 5, 10, 21),
    filter_zscore=20,
)

# Full tearsheet
al.tears.create_full_tear_sheet(factor_and_returns, long_short=True)
```

## Information Coefficient (IC)

```python
from scipy.stats import spearmanr
import pandas as pd

def rolling_ic(factor_series, forward_returns, window=None):
    """
    factor_series: pd.DataFrame (dates × assets), factor values
    forward_returns: pd.DataFrame (dates × assets), forward returns
    """
    ic_by_date = {}
    for date in factor_series.index:
        if date not in forward_returns.index:
            continue
        f = factor_series.loc[date].dropna()
        r = forward_returns.loc[date].dropna()
        common = f.index.intersection(r.index)
        if len(common) < 10:
            continue
        ic, _ = spearmanr(f[common], r[common])
        ic_by_date[date] = ic
    ic_series = pd.Series(ic_by_date)
    return ic_series

ic = rolling_ic(factor_df, forward_1d)
print(f"Mean IC: {ic.mean():.4f}")
print(f"IC IR (ICIR): {ic.mean() / ic.std():.2f}")
print(f"IC t-stat: {ic.mean() / (ic.std() / len(ic)**0.5):.2f}")
```

## Quantile Analysis

```python
def quantile_returns(factor_series, forward_returns, n_quantiles=5):
    quantiles = factor_series.apply(
        lambda row: pd.qcut(row, n_quantiles, labels=False, duplicates="drop"), axis=1
    )
    results = {}
    for q in range(n_quantiles):
        mask = quantiles == q
        q_ret = forward_returns[mask].mean(axis=1)
        results[f"Q{q+1}"] = q_ret
    qdf = pd.DataFrame(results)
    qdf["spread"] = qdf[f"Q{n_quantiles}"] - qdf["Q1"]
    return qdf
```

## Factor Neutralization

```python
import statsmodels.api as sm

def neutralize_factor(factor_series, neutralize_df):
    """
    Remove the component of factor explained by neutralize_df variables.
    Cross-sectional regression on each date.
    """
    residuals = {}
    for date in factor_series.index:
        y = factor_series.loc[date].dropna()
        X_raw = neutralize_df.loc[date].reindex(y.index).dropna()
        common = y.index.intersection(X_raw.index)
        if len(common) < 10:
            continue
        X = sm.add_constant(X_raw.loc[common])
        model = sm.OLS(y.loc[common], X).fit()
        residuals[date] = model.resid
    return pd.DataFrame(residuals).T

# Example: remove market beta from momentum factor
market_returns = prices.pct_change().mean(axis=1)  # equal-weight market
market_betas = # pre-computed beta DataFrame (dates × assets)
neutralized = neutralize_factor(momentum_factor, market_betas)
```

## Factor Decay

```python
def factor_decay(factor_series, prices, horizons=[1, 5, 10, 21]):
    ics = {}
    for h in horizons:
        fwd = prices.pct_change(h).shift(-h)
        ic = rolling_ic(factor_series, fwd)
        ics[h] = ic.mean()
    return pd.Series(ics, name="Mean IC")
```

## Links

- [alphalens-reloaded](https://github.com/stefan-jansen/alphalens-reloaded)
- [awesome-quant factor analysis section](https://github.com/wilsonfreitas/awesome-quant#factor-analysis)
