---
name: quant-backtesting
description: Design, implement, and evaluate trading strategy backtests using Backtrader, PyBroker, Zipline-reloaded, or vectorized pandas/numpy approaches
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Quant Backtesting Skill

Build rigorous, realistic backtests of trading strategies — from simple rule-based systems to ML-driven models — while avoiding common pitfalls like look-ahead bias and overfitting.

## When to Activate

- Implementing a new trading strategy and validating it on historical data
- Converting a research idea (alpha signal) into a backtest
- Comparing event-driven vs. vectorized backtesting approaches
- Diagnosing unrealistic backtest results (too-good equity curves)
- Adding transaction costs, slippage, or position sizing to an existing backtest

## Key Libraries

| Library | Style | Best For |
|---------|-------|----------|
| `backtrader` | Event-driven | Full-featured, flexible strategies |
| `PyBroker` | ML-first, event-driven | ML models integrated with execution |
| `zipline-reloaded` | Event-driven | Quantopian-style research |
| `vectorbt` | Vectorized | Fast parameter sweeps & optimization |
| pandas/numpy | Vectorized | Quick signal validation |

## Workflow

### Vectorized (fast prototyping)
```python
import pandas as pd
import numpy as np

# Assumes `prices` is a DataFrame of adjusted close prices
returns = prices.pct_change()

# Simple moving average crossover signal
fast = prices.rolling(20).mean()
slow = prices.rolling(50).mean()
signal = (fast > slow).astype(int).shift(1)  # shift(1) prevents look-ahead

strategy_returns = signal * returns
cumulative = (1 + strategy_returns).cumprod()
```

### Backtrader (event-driven)
```python
import backtrader as bt

class SmaCross(bt.Strategy):
    params = dict(fast=20, slow=50)

    def __init__(self):
        sma_fast = bt.ind.SMA(period=self.p.fast)
        sma_slow = bt.ind.SMA(period=self.p.slow)
        self.crossover = bt.ind.CrossOver(sma_fast, sma_slow)

    def next(self):
        if not self.position:
            if self.crossover > 0:
                self.buy()
        elif self.crossover < 0:
            self.close()

cerebro = bt.Cerebro()
cerebro.addstrategy(SmaCross)
cerebro.adddata(bt.feeds.PandasData(dataname=df))
cerebro.broker.setcash(100_000)
cerebro.broker.setcommission(commission=0.001)
results = cerebro.run()
cerebro.plot()
```

## Performance Metrics

Always compute these before trusting a backtest:

```python
import numpy as np

def backtest_stats(returns, rf=0.0, periods=252):
    ann_ret = returns.mean() * periods
    ann_vol = returns.std() * np.sqrt(periods)
    sharpe = (ann_ret - rf) / ann_vol
    cum = (1 + returns).cumprod()
    drawdown = (cum / cum.cummax() - 1)
    max_dd = drawdown.min()
    calmar = ann_ret / abs(max_dd)
    return {"ann_return": ann_ret, "ann_vol": ann_vol,
            "sharpe": sharpe, "max_drawdown": max_dd, "calmar": calmar}
```

## Common Pitfalls

| Pitfall | How to Avoid |
|---------|-------------|
| Look-ahead bias | Always `shift(1)` signals before multiplying by returns |
| Survivorship bias | Include delisted stocks in universe |
| Overfitting | Use walk-forward validation, not single train/test split |
| Ignoring costs | Model bid-ask spread + commission + market impact |
| Unrealistic fills | Use previous close or VWAP, not open of same bar |

## Walk-Forward Validation

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
for train_idx, test_idx in tscv.split(returns):
    train = returns.iloc[train_idx]
    test = returns.iloc[test_idx]
    # fit model on train, evaluate on test
```

## Links

- [Backtrader](https://github.com/mementum/backtrader)
- [PyBroker](https://github.com/edtechre/pybroker)
- [vectorbt](https://github.com/polakowo/vectorbt)
- [zipline-reloaded](https://github.com/stefan-jansen/zipline-reloaded)
- [awesome-quant backtesting section](https://github.com/wilsonfreitas/awesome-quant#trading--backtesting)
