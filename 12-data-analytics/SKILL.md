---
name: data-analytics
description: Auto-activating data analytics skill — A/B testing, anomaly detection, cohort analysis, funnel analysis, churn prediction, KPIs, SQL optimization, forecasting, and dashboards
origin: jeremylongshore/claude-code-plugins-plus-skills/12-data-analytics
tools: Read, Write, Edit, Bash, Grep
---

# Data Analytics Skill

Automated analytics assistance: statistical analysis, A/B testing, cohort and funnel analysis, SQL query optimization, KPI definition, forecasting, and visualization planning.

## When to Activate

Activates automatically when you mention: A/B test, cohort analysis, funnel, churn, retention, KPI, metric, anomaly detection, forecast, correlation, regression, SQL optimization, window function, CTE, pivot table, dashboard, visualization, data story.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `ab-test-analyzer` | Statistical significance, power, sample size calculation |
| `anomaly-detector` | Z-score, IQR, Prophet-based anomaly detection |
| `cohort-analysis-creator` | Cohort retention tables and heatmaps |
| `funnel-analysis-builder` | Conversion funnel with drop-off analysis |
| `churn-analysis-helper` | Churn rate, survival analysis, predictors |
| `retention-calculator` | D1/D7/D30 retention metrics |
| `correlation-analyzer` | Pearson/Spearman correlation matrices |
| `regression-analysis-helper` | OLS, logistic, polynomial regression |
| `statistical-significance-calculator` | t-test, chi-square, Mann-Whitney |
| `sql-query-optimizer` | Index hints, query rewrites, EXPLAIN analysis |
| `window-function-generator` | ROW_NUMBER, LEAD/LAG, running totals |
| `cte-query-builder` | Readable, performant CTE patterns |
| `aggregation-helper` | GROUP BY, HAVING, ROLLUP, CUBE |
| `forecast-generator` | Prophet, ARIMA forecasting with intervals |
| `time-series-decomposer` | Trend, seasonality, residual decomposition |
| `kpi-definition-helper` | KPI framework with numerator/denominator/target |
| `dashboard-layout-planner` | Dashboard wireframe and metric hierarchy |
| `chart-type-recommender` | Choose the right chart for each data type |
| `data-story-outliner` | Narrative structure for data presentations |
| `metric-calculator` | Common business metrics (LTV, CAC, NPS, ARPU) |

## A/B Test Analysis

```python
import numpy as np
from scipy import stats

def ab_test_analysis(control_conversions, control_visitors,
                     treatment_conversions, treatment_visitors,
                     alpha=0.05):
    p_control = control_conversions / control_visitors
    p_treatment = treatment_conversions / treatment_visitors

    # Two-proportion z-test
    p_pool = (control_conversions + treatment_conversions) / (control_visitors + treatment_visitors)
    se = np.sqrt(p_pool * (1 - p_pool) * (1/control_visitors + 1/treatment_visitors))
    z_stat = (p_treatment - p_control) / se
    p_value = 2 * (1 - stats.norm.cdf(abs(z_stat)))

    # Confidence interval
    ci_width = 1.96 * np.sqrt(p_control*(1-p_control)/control_visitors + p_treatment*(1-p_treatment)/treatment_visitors)
    lift = (p_treatment - p_control) / p_control

    return {
        "control_rate": round(p_control, 4),
        "treatment_rate": round(p_treatment, 4),
        "lift": f"{lift:.1%}",
        "z_statistic": round(z_stat, 3),
        "p_value": round(p_value, 4),
        "significant": p_value < alpha,
        "ci_lower": round((p_treatment - p_control) - ci_width, 4),
        "ci_upper": round((p_treatment - p_control) + ci_width, 4),
    }
```

## Cohort Retention SQL

```sql
WITH first_purchase AS (
    SELECT user_id, DATE_TRUNC('month', MIN(created_at)) AS cohort_month
    FROM orders GROUP BY 1
),
activity AS (
    SELECT o.user_id, fp.cohort_month,
           DATE_TRUNC('month', o.created_at) AS activity_month,
           DATEDIFF('month', fp.cohort_month, DATE_TRUNC('month', o.created_at)) AS period
    FROM orders o JOIN first_purchase fp USING (user_id)
),
cohort_size AS (
    SELECT cohort_month, COUNT(DISTINCT user_id) AS total_users
    FROM first_purchase GROUP BY 1
)
SELECT a.cohort_month, a.period, cs.total_users,
       COUNT(DISTINCT a.user_id) AS retained_users,
       ROUND(COUNT(DISTINCT a.user_id)::NUMERIC / cs.total_users, 3) AS retention_rate
FROM activity a
JOIN cohort_size cs USING (cohort_month)
GROUP BY 1, 2, 3
ORDER BY 1, 2
```

## Funnel Analysis

```python
import pandas as pd

def funnel_analysis(events_df: pd.DataFrame, steps: list[str], user_col="user_id") -> pd.DataFrame:
    funnel = []
    users_in_step = set(events_df[events_df["event"] == steps[0]][user_col].unique())
    funnel.append({"step": steps[0], "users": len(users_in_step), "conversion": 1.0})

    for i in range(1, len(steps)):
        next_users = set(
            events_df[
                (events_df["event"] == steps[i]) &
                (events_df[user_col].isin(users_in_step))
            ][user_col].unique()
        )
        conversion = len(next_users) / len(users_in_step) if users_in_step else 0
        funnel.append({"step": steps[i], "users": len(next_users), "conversion": conversion})
        users_in_step = next_users

    return pd.DataFrame(funnel)
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `12-data-analytics` (26 skills)
