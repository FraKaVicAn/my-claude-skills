---
name: quant-visualization
description: Visualize financial data with mplfinance candlestick charts, interactive Plotly/finplot dashboards, and D-Tale exploratory data analysis
origin: awesome-quant
tools: Read, Write, Edit, Bash
---

# Quant Visualization Skill

Create clear, publication-quality charts for financial data: OHLCV candlesticks, equity curves, drawdown plots, correlation heatmaps, and volatility surfaces.

## When to Activate

- Plotting candlestick charts with overlaid technical indicators
- Visualizing an equity curve with drawdown periods highlighted
- Building an interactive dashboard for a strategy or portfolio
- Creating a correlation heatmap or volatility surface plot
- Generating a multi-panel research report figure

## Key Libraries

| Library | Best For |
|---------|----------|
| `mplfinance` | Static OHLCV + indicator charts |
| `plotly` | Interactive web charts, dashboards |
| `finplot` | Fast interactive charts (desktop) |
| `matplotlib` | Custom subplots, publication figures |
| `seaborn` | Statistical/correlation heatmaps |
| `D-Tale` | Exploratory pandas DataFrame GUI |

## Candlestick Chart

```python
import mplfinance as mpf
import pandas as pd

# df must have OHLCV columns and DatetimeIndex
sma20 = df["Close"].rolling(20).mean()
sma50 = df["Close"].rolling(50).mean()

add_plots = [
    mpf.make_addplot(sma20, color="blue", width=1, label="SMA 20"),
    mpf.make_addplot(sma50, color="orange", width=1, label="SMA 50"),
    mpf.make_addplot(df["RSI"], panel=2, color="purple", ylabel="RSI"),
]

mpf.plot(df, type="candle", style="yahoo", addplot=add_plots,
         volume=True, title="AAPL — Daily", figratio=(14, 8),
         panel_ratios=(4, 1, 2), savefig="chart.png")
```

## Equity Curve + Drawdown

```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import numpy as np

def plot_equity_curve(returns, benchmark=None, title="Strategy"):
    cum = (1 + returns).cumprod()
    dd = (cum / cum.cummax() - 1)

    fig = plt.figure(figsize=(14, 8))
    gs = gridspec.GridSpec(2, 1, height_ratios=[3, 1], hspace=0.05)

    ax1 = fig.add_subplot(gs[0])
    ax1.plot(cum, label="Strategy", color="steelblue", linewidth=1.5)
    if benchmark is not None:
        bm_cum = (1 + benchmark).cumprod()
        ax1.plot(bm_cum, label="Benchmark", color="gray", linewidth=1, alpha=0.7)
    ax1.set_ylabel("Cumulative Return")
    ax1.legend()
    ax1.set_title(title)
    ax1.grid(alpha=0.3)

    ax2 = fig.add_subplot(gs[1], sharex=ax1)
    ax2.fill_between(dd.index, dd.values, 0, color="red", alpha=0.4, label="Drawdown")
    ax2.set_ylabel("Drawdown")
    ax2.grid(alpha=0.3)

    plt.tight_layout()
    return fig
```

## Interactive Plotly Chart

```python
import plotly.graph_objects as go
from plotly.subplots import make_subplots

def interactive_candlestick(df, indicators=None):
    fig = make_subplots(rows=2, cols=1, shared_xaxes=True,
                        vertical_spacing=0.03, row_heights=[0.7, 0.3])

    fig.add_trace(go.Candlestick(
        x=df.index, open=df["Open"], high=df["High"],
        low=df["Low"], close=df["Close"], name="Price"), row=1, col=1)

    fig.add_trace(go.Bar(
        x=df.index, y=df["Volume"], name="Volume",
        marker_color="rgba(100,100,200,0.4)"), row=2, col=1)

    if indicators:
        for name, series in indicators.items():
            fig.add_trace(go.Scatter(
                x=series.index, y=series.values, name=name, mode="lines"), row=1, col=1)

    fig.update_layout(xaxis_rangeslider_visible=False, height=600,
                      template="plotly_white", title=df.index[0].year)
    return fig

fig = interactive_candlestick(df, {"SMA 20": sma20, "SMA 50": sma50})
fig.show()
```

## Correlation Heatmap

```python
import seaborn as sns
import matplotlib.pyplot as plt

def plot_correlation(returns, method="pearson"):
    corr = returns.corr(method=method)
    mask = np.triu(np.ones_like(corr, dtype=bool))

    fig, ax = plt.subplots(figsize=(10, 8))
    sns.heatmap(corr, mask=mask, annot=True, fmt=".2f",
                cmap="RdYlGn", vmin=-1, vmax=1, center=0,
                square=True, linewidths=0.5, ax=ax)
    ax.set_title("Asset Return Correlations")
    return fig
```

## Volatility Surface

```python
from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt

def plot_vol_surface(vol_surface_df):
    """vol_surface_df: columns = T (maturity), moneyness, iv"""
    fig = plt.figure(figsize=(12, 7))
    ax = fig.add_subplot(111, projection="3d")
    ax.plot_trisurf(vol_surface_df["moneyness"], vol_surface_df["T"],
                    vol_surface_df["iv"], cmap="RdYlGn_r", alpha=0.8)
    ax.set_xlabel("Log-Moneyness")
    ax.set_ylabel("Maturity (years)")
    ax.set_zlabel("Implied Volatility")
    ax.set_title("Implied Volatility Surface")
    return fig
```

## D-Tale (exploratory)

```python
import dtale

# Launch interactive GUI in browser
d = dtale.show(df)
d.open_browser()
```

## Links

- [mplfinance](https://github.com/matplotlib/mplfinance)
- [finplot](https://github.com/highfestiva/finplot)
- [D-Tale](https://github.com/man-group/dtale)
- [awesome-quant visualization section](https://github.com/wilsonfreitas/awesome-quant#visualization)
