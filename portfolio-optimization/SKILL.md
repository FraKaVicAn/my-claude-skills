---
name: portfolio-optimization
description: Construct and optimize investment portfolios using PyPortfolioOpt, Riskfolio-Lib, or custom mean-variance / risk-parity approaches
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Portfolio Optimization Skill

Build optimal portfolios by maximizing risk-adjusted returns, minimizing volatility, or targeting specific risk allocations — using modern portfolio theory and robust alternatives.

## When to Activate

- Computing efficient frontier or maximum Sharpe ratio portfolio
- Implementing risk-parity, equal-weight, or minimum-volatility allocations
- Adding constraints (sector caps, turnover limits, long-only)
- Updating portfolio weights in a live rebalancing workflow
- Comparing classical Markowitz to robust/Black-Litterman alternatives

## Key Libraries

| Library | Strengths |
|---------|-----------|
| `PyPortfolioOpt` | Clean API, mean-variance, Black-Litterman, shrinkage |
| `Riskfolio-Lib` | 30+ risk measures, HRP, nested clustering |
| `scipy.optimize` | Custom objective functions |
| `cvxpy` | Convex optimization with arbitrary constraints |

## Workflow

### Mean-Variance (Max Sharpe)
```python
from pypfopt import EfficientFrontier, risk_models, expected_returns

mu = expected_returns.mean_historical_return(prices)
S = risk_models.CovarianceShrinkage(prices).ledoit_wolf()

ef = EfficientFrontier(mu, S)
ef.add_constraint(lambda w: w >= 0)          # long-only
ef.add_constraint(lambda w: w <= 0.25)       # max 25% per asset
weights = ef.max_sharpe()
cleaned = ef.clean_weights()
performance = ef.portfolio_performance(verbose=True)
```

### Minimum Volatility
```python
ef = EfficientFrontier(mu, S)
weights = ef.min_volatility()
```

### Hierarchical Risk Parity (HRP)
```python
from pypfopt import HRPOpt

returns = prices.pct_change().dropna()
hrp = HRPOpt(returns)
weights = hrp.optimize()
hrp.portfolio_performance(verbose=True)
```

### Black-Litterman
```python
from pypfopt import BlackLittermanModel, risk_models
from pypfopt.black_litterman import market_implied_risk_aversion

market_prices = prices["SPY"]
delta = market_implied_risk_aversion(market_prices)
S = risk_models.CovarianceShrinkage(prices).ledoit_wolf()

# Views: AAPL outperforms MSFT by 2%, GOOGL returns 5%
viewdict = {"AAPL": 0.02, "GOOGL": 0.05}
bl = BlackLittermanModel(S, pi="market", market_caps=mcaps,
                         risk_aversion=delta, absolute_views=viewdict)
bl_returns = bl.bl_returns()
ef = EfficientFrontier(bl_returns, S)
weights = ef.max_sharpe()
```

### Riskfolio-Lib (Risk Parity)
```python
import riskfolio as rp

port = rp.Portfolio(returns=returns)
port.assets_stats(method_mu="hist", method_cov="ledoit")
w = port.rp_optimization(model="Classic", rm="MV", rf=0, b=None)
```

## Constraints Reference

```python
ef = EfficientFrontier(mu, S)
# Sector neutrality
sector_mapper = {"AAPL": "Tech", "MSFT": "Tech", "JPM": "Finance"}
sector_upper = {"Tech": 0.4, "Finance": 0.3}
ef.add_sector_constraints(sector_mapper, sector_lower=None, sector_upper=sector_upper)

# Turnover limit (requires previous weights)
ef.add_constraint(lambda w: cp.sum(cp.abs(w - prev_w)) <= 0.10)
```

## Covariance Estimation

| Method | When to Use |
|--------|-------------|
| Sample covariance | Large T >> N |
| Ledoit-Wolf shrinkage | T ~ N (default choice) |
| Oracle approximating shrinkage (OAS) | N > T |
| Exponential weighted | Regime-aware, recent data weighted more |

```python
from pypfopt import risk_models
S = risk_models.CovarianceShrinkage(prices).ledoit_wolf()
S_exp = risk_models.exp_cov(prices, span=180)
```

## Links

- [PyPortfolioOpt](https://github.com/robertmartin8/PyPortfolioOpt)
- [Riskfolio-Lib](https://github.com/dcajasn/Riskfolio-Lib)
- [awesome-quant portfolio section](https://github.com/wilsonfreitas/awesome-quant#portfolio-optimization)
