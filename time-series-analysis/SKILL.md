---
name: time-series-analysis
description: Analyze financial time series with statsmodels, ARCH/GARCH volatility models, Prophet forecasting, and stationarity/cointegration tests
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Time Series Analysis Skill

Apply statistical time series methods to financial data: stationarity testing, ARIMA/GARCH modeling, volatility forecasting, cointegration, and pairs trading setup.

## When to Activate

- Testing if a price series is stationary or has a unit root
- Fitting ARIMA or SARIMA models to forecast returns or macro series
- Modeling and forecasting volatility with GARCH, EGARCH, or GJR-GARCH
- Testing for cointegration (pairs trading universe construction)
- Decomposing time series into trend, seasonality, and residual

## Key Libraries

| Library | Use Case |
|---------|----------|
| `statsmodels` | ARIMA, SARIMA, OLS, cointegration, ADF/KPSS tests |
| `arch` | ARCH/GARCH family volatility models |
| `prophet` | Trend + seasonality forecasting (macro, earnings) |
| `pyflux` | Bayesian time series |

## Stationarity & Unit Root Tests

```python
from statsmodels.tsa.stattools import adfuller, kpss

def test_stationarity(series, name=""):
    adf_stat, adf_p, _, _, adf_crit, _ = adfuller(series.dropna())
    kpss_stat, kpss_p, _, kpss_crit = kpss(series.dropna(), regression="c")
    print(f"{name} | ADF p={adf_p:.4f} | KPSS p={kpss_p:.4f}")
    # ADF: H0=unit root (want p<0.05 for stationarity)
    # KPSS: H0=stationary (want p>0.05 for stationarity)

test_stationarity(prices["AAPL"], "AAPL price")
test_stationarity(prices["AAPL"].pct_change().dropna(), "AAPL returns")
```

## ARIMA Modeling

```python
from statsmodels.tsa.arima.model import ARIMA
from pmdarima import auto_arima

# Auto-select orders
model = auto_arima(returns, seasonal=False, stepwise=True,
                   information_criterion="aic", trace=True)

# Fit and forecast
fitted = ARIMA(returns, order=model.order).fit()
forecast = fitted.forecast(steps=5)
conf_int = fitted.get_forecast(steps=5).conf_int()
```

## GARCH Volatility

```python
from arch import arch_model

# GARCH(1,1)
am = arch_model(returns * 100, vol="Garch", p=1, q=1, dist="normal")
res = am.fit(disp="off")
print(res.summary())

# Forecast volatility
forecast = res.forecast(horizon=5)
vol_forecast = forecast.variance.iloc[-1] ** 0.5 / 100  # back to decimal

# GJR-GARCH (asymmetric — captures leverage effect)
am_gjr = arch_model(returns * 100, vol="Garch", p=1, o=1, q=1)
res_gjr = am_gjr.fit(disp="off")
```

## Cointegration & Pairs Trading

```python
from statsmodels.tsa.stattools import coint
from statsmodels.regression.linear_model import OLS
import statsmodels.api as sm

# Test for cointegration
score, pvalue, _ = coint(prices["KO"], prices["PEP"])
print(f"Cointegration p-value: {pvalue:.4f}")  # want p < 0.05

# Estimate hedge ratio via OLS
X = sm.add_constant(prices["PEP"])
result = OLS(prices["KO"], X).fit()
hedge_ratio = result.params["PEP"]

# Spread
spread = prices["KO"] - hedge_ratio * prices["PEP"]
zscore = (spread - spread.mean()) / spread.std()

# Signal: enter when |z| > 2, exit when |z| < 0.5
long_signal = zscore < -2
short_signal = zscore > 2
```

## Prophet Forecasting

```python
from prophet import Prophet
import pandas as pd

df_prophet = prices[["Close"]].reset_index()
df_prophet.columns = ["ds", "y"]

model = Prophet(
    changepoint_prior_scale=0.05,
    yearly_seasonality=True,
    weekly_seasonality=True
)
model.add_seasonality(name="monthly", period=30.5, fourier_order=5)
model.fit(df_prophet)

future = model.make_future_dataframe(periods=252)
forecast = model.predict(future)
model.plot(forecast)
model.plot_components(forecast)
```

## Seasonal Decomposition

```python
from statsmodels.tsa.seasonal import seasonal_decompose, STL

# Classical decomposition
result = seasonal_decompose(prices["Close"], model="multiplicative", period=252)
result.plot()

# STL (robust to outliers)
stl = STL(prices["Close"], period=252, robust=True)
res = stl.fit()
res.plot()
```

## Links

- [arch](https://github.com/bashtage/arch)
- [statsmodels](https://github.com/statsmodels/statsmodels)
- [Prophet](https://github.com/facebook/prophet)
- [pmdarima](https://github.com/alkaline-ml/pmdarima)
- [awesome-quant time series section](https://github.com/wilsonfreitas/awesome-quant#time-series)
