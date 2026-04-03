---
name: risk-analysis
description: Compute portfolio risk metrics — VaR, CVaR/ES, drawdown, beta, tracking error, Sharpe — using historical simulation, parametric, or Monte Carlo methods
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Risk Analysis Skill

Measure and decompose portfolio risk using standard risk metrics for reporting, position sizing, or regulatory purposes.

## When to Activate

- Computing Value at Risk (VaR) or Expected Shortfall (CVaR/ES)
- Analyzing drawdown profile of a strategy or portfolio
- Decomposing portfolio risk by asset, factor, or sector
- Calculating beta, tracking error, or information ratio vs. a benchmark
- Performing stress tests or scenario analysis

## Key Libraries

| Library | Use Case |
|---------|----------|
| `Riskfolio-Lib` | CVaR, CDaR, tail risk, factor models |
| `PyPortfolioOpt` | Risk contribution, volatility measures |
| `statsmodels` | Beta estimation, OLS factor models |
| `scipy.stats` | Parametric VaR |
| `empyrical` | Zipline-compatible performance/risk stats |

## Value at Risk

```python
import numpy as np
import pandas as pd
from scipy import stats

def var_historical(returns, confidence=0.95):
    """Historical simulation VaR (negative = loss)."""
    return -np.percentile(returns, (1 - confidence) * 100)

def var_parametric(returns, confidence=0.95):
    """Assumes normal distribution."""
    mu, sigma = returns.mean(), returns.std()
    return -(mu + stats.norm.ppf(1 - confidence) * sigma)

def cvar(returns, confidence=0.95):
    """Expected Shortfall (average loss beyond VaR)."""
    var = var_historical(returns, confidence)
    return -returns[returns < -var].mean()

# Example
daily_var_95 = var_historical(portfolio_returns, 0.95)
daily_es_95 = cvar(portfolio_returns, 0.95)
print(f"1-day 95% VaR: {daily_var_95:.2%}")
print(f"1-day 95% ES:  {daily_es_95:.2%}")

# Scale to 10-day (regulatory): VaR * sqrt(10)
var_10d = daily_var_95 * np.sqrt(10)
```

## Monte Carlo VaR

```python
def mc_var(returns, n_simulations=10_000, horizon=1, confidence=0.95):
    mu = returns.mean()
    sigma = returns.std()
    sims = np.random.normal(mu * horizon, sigma * np.sqrt(horizon), n_simulations)
    return -np.percentile(sims, (1 - confidence) * 100)
```

## Drawdown Analysis

```python
def drawdown_series(returns):
    cum = (1 + returns).cumprod()
    rolling_max = cum.cummax()
    return (cum / rolling_max) - 1

def drawdown_stats(returns):
    dd = drawdown_series(returns)
    max_dd = dd.min()
    # Duration: longest drawdown period
    in_drawdown = dd < 0
    transitions = in_drawdown.astype(int).diff().fillna(0)
    starts = transitions[transitions == 1].index
    ends = transitions[transitions == -1].index
    if len(starts) and len(ends):
        durations = [(e - s).days for s, e in zip(starts, ends[:len(starts)])]
        max_duration = max(durations)
    else:
        max_duration = None
    return {"max_drawdown": max_dd, "max_duration_days": max_duration,
            "current_drawdown": dd.iloc[-1]}
```

## Beta & Factor Exposure

```python
import statsmodels.api as sm

def compute_beta(asset_returns, benchmark_returns):
    X = sm.add_constant(benchmark_returns)
    model = sm.OLS(asset_returns, X).fit()
    alpha, beta = model.params
    return {"alpha_annualized": alpha * 252, "beta": beta,
            "r_squared": model.rsquared, "t_stat_beta": model.tvalues["x1"]}

# Fama-French style multi-factor
def factor_model(portfolio_returns, factor_df):
    """factor_df: columns are factor returns (MKT, SMB, HML, etc.)"""
    X = sm.add_constant(factor_df)
    model = sm.OLS(portfolio_returns, X).fit()
    return model.params, model.rsquared
```

## Performance Summary

```python
def performance_summary(returns, benchmark=None, rf=0.0, periods=252):
    ann_ret = returns.mean() * periods
    ann_vol = returns.std() * np.sqrt(periods)
    sharpe = (ann_ret - rf) / ann_vol
    dd = drawdown_series(returns)
    max_dd = dd.min()
    calmar = ann_ret / abs(max_dd) if max_dd != 0 else np.nan

    result = {
        "Ann. Return": f"{ann_ret:.2%}",
        "Ann. Volatility": f"{ann_vol:.2%}",
        "Sharpe Ratio": f"{sharpe:.2f}",
        "Max Drawdown": f"{max_dd:.2%}",
        "Calmar Ratio": f"{calmar:.2f}",
    }

    if benchmark is not None:
        beta_info = compute_beta(returns, benchmark)
        excess = returns - benchmark
        te = excess.std() * np.sqrt(periods)
        ir = excess.mean() * periods / te
        result.update({
            "Beta": f"{beta_info['beta']:.2f}",
            "Alpha (ann.)": f"{beta_info['alpha_annualized']:.2%}",
            "Tracking Error": f"{te:.2%}",
            "Information Ratio": f"{ir:.2f}",
        })
    return pd.Series(result)
```

## Links

- [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib)
- [empyrical](https://github.com/quantopian/empyrical)
- [awesome-quant risk section](https://github.com/wilsonfreitas/awesome-quant#risk-analysis)
