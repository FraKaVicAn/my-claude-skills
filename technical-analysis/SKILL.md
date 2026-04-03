---
name: technical-analysis
description: Compute and apply technical indicators (moving averages, RSI, MACD, Bollinger Bands, etc.) using TA-Lib, pandas-ta, or custom implementations
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Technical Analysis Skill

Add technical indicators to price data for signal generation, strategy development, or exploratory analysis.

## When to Activate

- Computing standard indicators (SMA, EMA, RSI, MACD, Bollinger Bands, ATR)
- Building a feature set for an ML trading model
- Generating buy/sell signals from indicator crossovers or threshold breaches
- Auditing or replicating an existing indicator-based strategy

## Key Libraries

| Library | Notes |
|---------|-------|
| `ta-lib` | C-backed, fast, 150+ indicators; requires binary install |
| `pandas-ta` | Pure Python, pandas-native, 130+ indicators |
| `tulipy` | Tulip Indicators binding, lightweight |
| `finta` | Simple pandas-based indicators |

## Common Indicators

### Trend
```python
import pandas_ta as ta

df["SMA_20"] = ta.sma(df["Close"], length=20)
df["EMA_20"] = ta.ema(df["Close"], length=20)
df["WMA_20"] = ta.wma(df["Close"], length=20)

macd = ta.macd(df["Close"], fast=12, slow=26, signal=9)
df = df.join(macd)
```

### Momentum
```python
df["RSI"] = ta.rsi(df["Close"], length=14)
df["ROC"] = ta.roc(df["Close"], length=10)
stoch = ta.stoch(df["High"], df["Low"], df["Close"])
df = df.join(stoch)
```

### Volatility
```python
bbands = ta.bbands(df["Close"], length=20, std=2)
df = df.join(bbands)

df["ATR"] = ta.atr(df["High"], df["Low"], df["Close"], length=14)
```

### Volume
```python
df["OBV"] = ta.obv(df["Close"], df["Volume"])
df["VWAP"] = ta.vwap(df["High"], df["Low"], df["Close"], df["Volume"])
```

### Using TA-Lib directly
```python
import talib
import numpy as np

close = df["Close"].values.astype(float)
df["RSI"] = talib.RSI(close, timeperiod=14)
macd, signal, hist = talib.MACD(close, fastperiod=12, slowperiod=26, signalperiod=9)
upper, middle, lower = talib.BBANDS(close, timeperiod=20, nbdevup=2, nbdevdn=2)
```

## Signal Generation

```python
# RSI overbought/oversold
df["rsi_signal"] = 0
df.loc[df["RSI"] < 30, "rsi_signal"] = 1   # oversold → buy
df.loc[df["RSI"] > 70, "rsi_signal"] = -1  # overbought → sell

# MACD crossover
df["macd_signal"] = np.where(
    df["MACD_12_26_9"] > df["MACDs_12_26_9"], 1, -1
)

# Bollinger Band squeeze entry
df["bb_signal"] = np.where(df["Close"] < df["BBL_20_2.0"], 1,
                  np.where(df["Close"] > df["BBU_20_2.0"], -1, 0))
```

## ML Feature Engineering

```python
feature_cols = []
for length in [5, 10, 20, 50]:
    df[f"sma_{length}"] = ta.sma(df["Close"], length=length)
    df[f"roc_{length}"] = ta.roc(df["Close"], length=length)
    feature_cols += [f"sma_{length}", f"roc_{length}"]

df["rsi_14"] = ta.rsi(df["Close"], length=14)
df["atr_14"] = ta.atr(df["High"], df["Low"], df["Close"], length=14)
feature_cols += ["rsi_14", "atr_14"]

X = df[feature_cols].dropna()
```

## Installation Notes

TA-Lib requires the underlying C library:
```bash
# macOS
brew install ta-lib && pip install TA-Lib

# Windows — use a pre-built wheel
pip install TA_Lib‑0.4.28‑cp311‑cp311‑win_amd64.whl

# Linux
apt-get install libta-lib-dev && pip install TA-Lib
```

pandas-ta is easier:
```bash
pip install pandas-ta
```

## Links

- [TA-Lib](https://github.com/TA-Lib/ta-lib-python)
- [pandas-ta](https://github.com/twopirllc/pandas-ta)
- [awesome-quant technical analysis section](https://github.com/wilsonfreitas/awesome-quant#technical-indicators)
