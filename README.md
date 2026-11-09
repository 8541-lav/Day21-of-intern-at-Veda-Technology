# A/B Test Exploratory Analysis

**Task 21 · Data Analytics Track**

Compare the conversion rates of two webpage variants (A = control, B = variant), test whether the difference is statistically significant, quantify its size and uncertainty, and give a rollout recommendation.

## Objective

Understand practical A/B test interpretation: state hypotheses, run the test, report effect size with confidence intervals, and distinguish statistical from practical significance.

## Dataset

**File:** `ab_testing.csv` (5,000 users, 7 columns)

| Column | Description |
|---|---|
| `User ID` | Unique identifier for each user |
| `Group` | Experiment arm: `A` (control) or `B` (variant) |
| `Page Views` | Number of pages viewed in the session |
| `Time Spent` | Time on site, in seconds |
| `Conversion` | Whether the user converted (`Yes` / `No`) |
| `Device` | `Desktop` or `Mobile` |
| `Location` | England, Northern Ireland, Scotland or Wales |

Data quality: no missing values, no duplicate User IDs, and no sample ratio mismatch (χ² = 0.29, p = 0.59).

## Hypotheses

- **H0:** p_B = p_A (no difference in conversion rate)
- **H1:** p_B ≠ p_A (two-sided, α = 0.05)

## Methodology

1. Data quality checks (missing values, duplicates, sample ratio mismatch)
2. Two-proportion z-test, cross-checked with chi-square and Fisher's exact test
3. 95% confidence interval for the difference in proportions
4. Effect sizes: absolute lift, relative lift, risk ratio, odds ratio, Cohen's h
5. Welch t-tests on secondary metrics (page views, time spent)
6. Exploratory segment breakdown by device and location

## Comparison Table

| Metric | A (Control) | B (Variant) | Difference |
|---|---|---|---|
| Users | 2,519 | 2,481 | -38 |
| Conversions | 136 | 349 | +213 |
| **Conversion rate** | **5.40%** | **14.07%** | **+8.67 pp** |
| Avg. page views | 7.58 | 7.49 | -0.09 (p = 0.44) |
| Avg. time spent (s) | 241.7 | 243.3 | +1.6 (p = 0.64) |

## Test Result

| Statistic | Value |
|---|---|
| Test | Two-proportion z-test (pooled) |
| z-statistic | 10.35 |
| p-value | < 0.001 |
| Absolute lift | +8.67 pp (95% CI: 7.04 to 10.30 pp) |
| Relative lift | +160.5% (95% CI: about +115% to +215%) |
| Risk ratio | 2.61 |
| Odds ratio | 2.87 |
| Cohen's h | 0.30 (small to medium) |

**Conclusion:** reject H0. Variant B has a significantly higher conversion rate, and the lift is large in practical terms.

## Segment Results (exploratory)

| Segment | A rate | B rate | Lift (pp) |
|---|---|---|---|
| Desktop | 5.87% | 13.91% | +8.04 |
| Mobile | 4.94% | 14.24% | +9.30 |
| England | 6.93% | 14.69% | +7.76 |
| Northern Ireland | 5.05% | 11.47% | +6.42 |
| Scotland | 4.93% | 15.06% | +10.13 |
| Wales | 4.77% | 15.12% | +10.35 |

B outperforms A in every segment. Each segment has only a few hundred users per arm, so differences between segments should not be over-interpreted.

## Recommendation

**Roll out Variant B**, ideally in stages with a small holdout, and monitor conversion, page views and time spent after launch. Because the effect is unusually large, first verify that tracking and randomisation were identical in both arms.

## Statistical vs Practical Significance

Statistical significance shows a difference is unlikely to be due to chance and depends heavily on sample size. Practical significance asks whether the difference is big enough to matter. Here both hold: p < 0.001 and a lift of about 9 extra conversions per 100 users, with the lower confidence bound still around 7 pp.

## Why Use a Control Group?

The control group provides the baseline needed to isolate the effect of the change. Without it, seasonality, marketing or other external factors could be mistaken for the variant's effect. Random assignment balances those factors across groups, supporting a causal interpretation.

## Tools

- Python (pandas, NumPy, SciPy, Matplotlib)
- Excel (comparison table, PivotTable of Conversion by Group)

## How to Reproduce

```bash
pip install pandas numpy scipy matplotlib
```

```python
import pandas as pd, numpy as np
from scipy import stats

df = pd.read_csv("ab_testing.csv")
df["converted"] = (df["Conversion"] == "Yes").astype(int)

g = df.groupby("Group")["converted"].agg(["size", "sum", "mean"])
a, b = g.loc["A"], g.loc["B"]

diff = b["mean"] - a["mean"]
se = np.sqrt(a["mean"]*(1-a["mean"])/a["size"] + b["mean"]*(1-b["mean"])/b["size"])
print(f"Lift: {diff:.4f}  95% CI: [{diff-1.96*se:.4f}, {diff+1.96*se:.4f}]")

p = (a["sum"] + b["sum"]) / (a["size"] + b["size"])
z = diff / np.sqrt(p*(1-p)*(1/a["size"] + 1/b["size"]))
print(f"z = {z:.2f}, p = {2*(1-stats.norm.cdf(abs(z))):.2e}")

h = 2*np.arcsin(np.sqrt(b["mean"])) - 2*np.arcsin(np.sqrt(a["mean"]))
print(f"Cohen's h = {h:.2f}")
```

## Limitations

- No timestamps, so novelty effects and test duration can't be assessed
- No revenue data; conversion is a binary flag
- Randomisation process is not documented
- Very large effect size should be validated against tracking issues

## Author

Lav — Data Analytics Track, Task 21
