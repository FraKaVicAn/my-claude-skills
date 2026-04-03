---
name: csv-data-summarizer
description: Analyzes CSV files, generates summary stats, and plots quick visualizations using Python and pandas. Auto-activates when user uploads or references a CSV file.
origin: coffeefuelbump/csv-data-summarizer-claude-skill
tools: Read, Write, Edit, Bash
---

# CSV Data Summarizer

Automatically analyzes CSV files and generates comprehensive summaries with statistical insights and visualizations — no questions asked.

## CRITICAL BEHAVIOR

**DO NOT ASK THE USER WHAT THEY WANT TO DO WITH THE DATA.**

When a CSV file is provided, IMMEDIATELY AND AUTOMATICALLY:
1. Run the comprehensive analysis
2. Generate ALL relevant visualizations
3. Present complete results

## How to Analyze

```python
# Run via analyze.py if available, or use inline:
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv(file_path)
```

### Step-by-step analysis:

1. **Load and inspect** — shape, column names, dtypes
2. **Data quality** — missing values by column
3. **Numeric analysis** — describe(), correlations if multiple numeric cols
4. **Categorical analysis** — value_counts() for object columns (skip ID cols)
5. **Time series** — if date/time columns exist, plot trends over time
6. **Visualizations** — only create charts that are relevant to the data:
   - Correlation heatmap → only if 2+ numeric columns
   - Time-series plots → only if date/timestamp columns exist
   - Distribution histograms → for numeric columns
   - Categorical bar charts → for categorical columns

### Adaptive detection by data type:
- **Sales/E-commerce** (order dates, revenue, products): Time-series trends, revenue analysis, product performance
- **Customer data** (demographics, segments): Distribution analysis, segmentation patterns
- **Financial** (transactions, amounts): Trend analysis, statistical summaries, correlations
- **Survey** (categorical responses, ratings): Frequency analysis, cross-tabulations
- **Generic**: Adapt based on column types found

## Core Analysis Script

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

def summarize_csv(file_path):
    df = pd.read_csv(file_path)
    
    print(f"Shape: {df.shape[0]:,} rows × {df.shape[1]} columns")
    print(f"Columns: {', '.join(df.columns.tolist())}")
    
    # Data quality
    missing = df.isnull().sum()
    if missing.any():
        print("\nMissing values:")
        for col, n in missing[missing > 0].items():
            print(f"  {col}: {n} ({n/len(df)*100:.1f}%)")
    else:
        print("\n✓ No missing values")
    
    # Numeric summary
    numeric_cols = df.select_dtypes(include='number').columns.tolist()
    if numeric_cols:
        print("\nNumeric summary:")
        print(df[numeric_cols].describe())
    
    # Correlations
    if len(numeric_cols) > 1:
        corr = df[numeric_cols].corr()
        plt.figure(figsize=(10, 8))
        sns.heatmap(corr, annot=True, cmap='coolwarm', center=0)
        plt.title('Correlation Heatmap')
        plt.tight_layout()
        plt.savefig('correlation_heatmap.png', dpi=150)
        plt.close()
    
    # Time series
    date_cols = [c for c in df.columns if 'date' in c.lower() or 'time' in c.lower()]
    if date_cols and numeric_cols:
        date_col = date_cols[0]
        df[date_col] = pd.to_datetime(df[date_col], errors='coerce')
        fig, axes = plt.subplots(min(3, len(numeric_cols)), 1,
                                 figsize=(12, 4 * min(3, len(numeric_cols))))
        if len(numeric_cols) == 1:
            axes = [axes]
        for idx, col in enumerate(numeric_cols[:3]):
            df.groupby(date_col)[col].mean().plot(ax=axes[idx], linewidth=2)
            axes[idx].set_title(f'{col} Over Time')
            axes[idx].grid(True, alpha=0.3)
        plt.tight_layout()
        plt.savefig('time_series.png', dpi=150)
        plt.close()
    
    # Distributions
    if numeric_cols:
        fig, axes = plt.subplots(2, 2, figsize=(12, 10))
        for idx, col in enumerate(numeric_cols[:4]):
            axes.flatten()[idx].hist(df[col].dropna(), bins=30, alpha=0.7)
            axes.flatten()[idx].set_title(f'Distribution: {col}')
        plt.tight_layout()
        plt.savefig('distributions.png', dpi=150)
        plt.close()

summarize_csv('your_file.csv')
```

## Dependencies

```
pip install pandas>=2.0.0 matplotlib>=3.7.0 seaborn>=0.12.0
```

## Example Prompts (auto-triggers)

- Upload or reference any `.csv` file
- "Analyze this CSV"
- "Summarize sales_data.csv"
- "What insights are in orders.csv?"
- "Show me trends in this data"
