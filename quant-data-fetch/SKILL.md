---
name: quant-data-fetch
description: Fetch and prepare financial market data using yfinance, pandas-datareader, OpenBB, or Nasdaq Data Link for quant analysis
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Quant Data Fetch Skill

Retrieve, clean, and structure financial market data from standard sources for quantitative analysis pipelines.

## When to Activate

- Downloading OHLCV price data for stocks, ETFs, indices, or crypto
- Fetching fundamental data (earnings, dividends, balance sheets)
- Pulling options chains, futures curves, or FX rates
- Building a data pipeline for backtesting or research notebooks
- Switching between data providers (yfinance → OpenBB → Nasdaq Data Link)

## Key Libraries

| Library | Use Case |
|---------|----------|
| `yfinance` | Free Yahoo Finance OHLCV, dividends, splits |
| `pandas-datareader` | FRED macro data, Stooq, IEX |
| `OpenBB` | Unified multi-source terminal SDK |
| `nasdaq-data-link` (Quandl) | Premium datasets |
| `exchange_calendars` | Market open/close schedules |
| `bizdays` | Business day calculations |

## Workflow

### 1. Single Ticker Download
```python
import yfinance as yf

ticker = yf.Ticker("AAPL")
df = ticker.history(start="2020-01-01", end="2024-12-31")
df.index = df.index.tz_localize(None)  # strip timezone for clean indexing
```

### 2. Multiple Tickers (Panel Data)
```python
import yfinance as yf

tickers = ["AAPL", "MSFT", "GOOGL", "SPY"]
prices = yf.download(tickers, start="2020-01-01", auto_adjust=True)["Close"]
returns = prices.pct_change().dropna()
```

### 3. Macro Data via pandas-datareader
```python
import pandas_datareader as pdr

# FRED series: 10Y Treasury yield, CPI, unemployment
fred_series = {
    "DGS10": "10Y_yield",
    "CPIAUCSL": "CPI",
    "UNRATE": "unemployment"
}
macro = {name: pdr.get_data_fred(code, start="2000-01-01")
         for code, name in fred_series.items()}
```

### 4. Options Chain
```python
import yfinance as yf

ticker = yf.Ticker("AAPL")
exp_dates = ticker.options
chain = ticker.option_chain(exp_dates[0])
calls = chain.calls
puts = chain.puts
```

### 5. Respecting Market Calendars
```python
import exchange_calendars as xcals

nyse = xcals.get_calendar("XNYS")
sessions = nyse.sessions_in_range("2024-01-01", "2024-12-31")
# Use sessions as valid trading day index
```

## Best Practices

- **Cache raw data locally** (Parquet/HDF5) to avoid re-downloading; APIs rate-limit
- **Use `auto_adjust=True`** in yfinance to get split/dividend-adjusted prices by default
- **Align all series to a common calendar** before joining — different assets have different trading days
- **Store with timezone-aware or consistently stripped timestamps** — mixing causes silent alignment bugs
- **Validate continuity**: check for gaps > 1 business day in daily data

## Common Issues

| Issue | Fix |
|-------|-----|
| Timezone mismatch | `.tz_localize(None)` or `.tz_convert("UTC")` uniformly |
| Survivorship bias | Download from a point-in-time database or use delisted tickers explicitly |
| Adjusted vs. unadjusted | Always document which price series you're using |
| Missing data | Forward-fill sparingly; prefer explicit `dropna()` for returns |

## Links

- [yfinance](https://github.com/ranaroussi/yfinance)
- [pandas-datareader](https://github.com/pydata/pandas-datareader)
- [OpenBB](https://github.com/OpenBB-finance/OpenBBTerminal)
- [exchange_calendars](https://github.com/rsheftel/pandas_market_calendars)
- [awesome-quant market data section](https://github.com/wilsonfreitas/awesome-quant#data-sources)
