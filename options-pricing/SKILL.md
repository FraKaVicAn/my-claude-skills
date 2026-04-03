---
name: options-pricing
description: Price and analyze options and derivatives using QuantLib, FinancePy, vollib, or closed-form Black-Scholes implementations
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Options Pricing Skill

Price vanilla and exotic options, compute Greeks, build volatility surfaces, and analyze derivatives positions.

## When to Activate

- Computing Black-Scholes option prices and Greeks
- Building an implied volatility surface from market quotes
- Pricing path-dependent or exotic options (barriers, Asian, lookback)
- Calculating P&L attribution for options portfolios
- Fitting volatility models (SVI, SABR, Heston)

## Key Libraries

| Library | Strengths |
|---------|-----------|
| `QuantLib` | Industrial-grade, full yield curves, many models |
| `FinancePy` | Modern Python-first QuantLib alternative |
| `vollib` | Black-Scholes, Black-76, LetsBeRational (fast IV) |
| `py_vollib_vectorized` | Vectorized Greeks for large option chains |

## Black-Scholes Closed Form

```python
from scipy.stats import norm
import numpy as np

def black_scholes(S, K, T, r, sigma, option_type="call"):
    """S=spot, K=strike, T=years to expiry, r=risk-free rate, sigma=vol"""
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)
    if option_type == "call":
        return S * norm.cdf(d1) - K * np.exp(-r * T) * norm.cdf(d2)
    return K * np.exp(-r * T) * norm.cdf(-d2) - S * norm.cdf(-d1)

def greeks(S, K, T, r, sigma, option_type="call"):
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)
    delta = norm.cdf(d1) if option_type == "call" else norm.cdf(d1) - 1
    gamma = norm.pdf(d1) / (S * sigma * np.sqrt(T))
    theta = (-(S * norm.pdf(d1) * sigma) / (2 * np.sqrt(T))
             - r * K * np.exp(-r * T) * norm.cdf(d2 if option_type == "call" else -d2))
    vega = S * norm.pdf(d1) * np.sqrt(T)
    return {"delta": delta, "gamma": gamma, "theta": theta / 365, "vega": vega / 100}
```

## Implied Volatility

```python
from scipy.optimize import brentq

def implied_vol(market_price, S, K, T, r, option_type="call"):
    objective = lambda sigma: black_scholes(S, K, T, r, sigma, option_type) - market_price
    try:
        return brentq(objective, 1e-6, 10.0, xtol=1e-6)
    except ValueError:
        return np.nan

# Fast vectorized IV using vollib
from py_vollib_vectorized import vectorized_implied_volatility as viv
ivs = viv(prices, S, strikes, T, r, flag="c", q=0, return_as="numpy")
```

## QuantLib Pricing

```python
import QuantLib as ql

# Setup
today = ql.Date.todaysDate()
ql.Settings.instance().evaluationDate = today
S, K, r, sigma, T_days = 100, 105, 0.05, 0.20, 90

spot = ql.SimpleQuote(S)
rate = ql.SimpleQuote(r)
vol = ql.SimpleQuote(sigma)

riskfree_ts = ql.YieldTermStructureHandle(
    ql.FlatForward(today, ql.QuoteHandle(rate), ql.Actual365Fixed()))
vol_ts = ql.BlackVolTermStructureHandle(
    ql.BlackConstantVol(today, ql.NullCalendar(), ql.QuoteHandle(vol), ql.Actual365Fixed()))

process = ql.BlackScholesProcess(
    ql.QuoteHandle(spot), riskfree_ts, vol_ts)

expiry = today + T_days
exercise = ql.EuropeanExercise(expiry)
payoff = ql.PlainVanillaPayoff(ql.Option.Call, K)
option = ql.VanillaOption(payoff, exercise)
option.setPricingEngine(ql.AnalyticEuropeanEngine(process))

print(f"Price: {option.NPV():.4f}")
print(f"Delta: {option.delta():.4f}")
print(f"Gamma: {option.gamma():.4f}")
print(f"Vega:  {option.vega():.4f}")
```

## Volatility Surface

```python
import pandas as pd
import numpy as np

def build_vol_surface(chain_df, S, r, today):
    """chain_df: columns = strike, expiry_date, mid_price, option_type"""
    records = []
    for _, row in chain_df.iterrows():
        T = (row["expiry_date"] - today).days / 365
        if T <= 0:
            continue
        iv = implied_vol(row["mid_price"], S, row["strike"], T, r, row["option_type"])
        moneyness = np.log(row["strike"] / S) / np.sqrt(T)
        records.append({"T": T, "moneyness": moneyness, "iv": iv})
    return pd.DataFrame(records).dropna()
```

## Links

- [QuantLib-Python](https://github.com/lballabio/QuantLib)
- [FinancePy](https://github.com/domokane/FinancePy)
- [vollib](https://github.com/vollib/vollib)
- [py_vollib_vectorized](https://github.com/marcdemers/py_vollib_vectorized)
- [awesome-quant options section](https://github.com/wilsonfreitas/awesome-quant#financial-instruments-and-pricing)
