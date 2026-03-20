# Trader Performance vs Market Sentiment
## Overview

This project analyzes how Bitcoin market sentiment (Fear/Greed Index) relates to trader behavior and performance on the Hyperliquid derivatives exchange. The analysis covers 211,224 trades across 32 accounts from May 2023 to May 2025.

---

## Repository Structure

```
TRADERSENTIMENT_ANALYSIS/
├── data/
│   ├── fear_greed_index.csv        # Bitcoin Fear/Greed Index (2018–2025)
│   └── historical_data.csv         # Hyperliquid trader data
├── docs/
│   ├── Data Science Intern Assignment.md   # Original assignment brief
│   └── summary.md                  # One-page write-up (methodology, insights, strategy)
├── images/
│   └── part_b_charts.png           # Output charts from Part B analysis
├── notebook.ipynb                  # Main analysis notebook (Parts A, B, C + Bonus)
└── README.md                       # This file
```

---

## Setup

### Requirements

```
Python 3.9+
pandas
numpy
matplotlib
scikit-learn
jupyter
```

### Install dependencies

```bash
pip install pandas numpy matplotlib scikit-learn jupyter
```

### Run the notebook

1. Clone the repository
2. Place both CSV files inside the `data/` folder
3. Launch Jupyter:

```bash
jupyter notebook notebook.ipynb
```

4. Run all cells in order from top to bottom  **do not skip cells**, each section builds on the previous one
5. Outputs (charts, printed tables) will appear inline below each cell

Each section is clearly labelled in the notebook:

```
Part A — Section 1 : Load Fear/Greed data
Part A — Section 2 : Load trader data
Part A — Section 3 : Diagnose Unix timestamp
Part A — Section 4 : Parse timestamps and extract date
Part A — Section 5 : Merge datasets
Part A — Section 6 : Feature engineering and metrics
Part B — Question 1 : Performance by sentiment
Part B — Question 2 : Behavior by sentiment
Part B — Question 3 : Trader segmentation
Part B — Charts
Part C — Strategy recommendations
Bonus  — Predictive model
```

---

## Datasets

### 1. Bitcoin Fear/Greed Index (`data/fear_greed_index.csv`)

- 2,645 daily rows from 2018-02-01 to 2025-05-02
- Columns: `timestamp`, `value` (0–100), `classification`, `date`
- 5 sentiment categories: Extreme Fear, Fear, Neutral, Greed, Extreme Greed
- No missing values or duplicates

### 2. Hyperliquid Trader Data (`data/historical_data.csv`)

- 211,224 trade rows across 32 unique accounts
- Date range: 2023-05-01 to 2025-05-01 (480 unique trading days)
- 246 unique coins traded
- Key columns: `Account`, `Coin`, `Execution Price`, `Size USD`, `Side`, `Timestamp IST`, `Closed PnL`, `Fee`

---

## Data Issues Found and Resolved

| Issue | Resolution |
|---|---|
| Unix `Timestamp` column corrupted by Excel scientific notation  only 7 unique values across 211,224 rows | Column dropped. `Timestamp IST` used as sole time reference |
| `2024-10-26` missing from Fear/Greed index | Forward-filled from Oct 25 sentiment (Greed, value=72) — standard practice for single-day gaps |
| 50.6% of trades have `Closed PnL = 0` (open positions, not break-even trades) | Retained. Win rate calculated on closed trades only throughout |
| 344 rows from 2 accounts pre-dating 2024 | Retained — legitimate trades, covered by Fear/Greed index, 0.16% of data |

---

## Part A — Data Preparation Summary

- Both datasets loaded, profiled, and documented (shape, dtypes, nulls, duplicates)
- Timestamps parsed using explicit format string to avoid day/month ambiguity
- Left merge on `date` column 100% sentiment coverage after forward fill
- Helper columns added: `is_closed`, `is_win_closed`, `is_long`
- Daily aggregation per account: `daily_pnl`, `trade_count`, `win_rate`, `long_ratio`, `avg_size_usd`, `total_fees`, `drawdown`
- Account-level summary built: `total_pnl`, `pnl_per_day`, `trades_per_day`, `activity_rate`, `overall_win_rate`
- Three trader segments defined from verified data distributions

---

## Part B — Key Findings

### Q1: Does performance differ between Fear and Greed days?

- Median daily PnL is higher on Greed days ($267) than Fear days ($123)  2.2x difference
- Mean PnL is higher on Fear days ($5,185) but driven by outlier days, not consistent performance
- Win rate is nearly identical across sentiment (Fear: 84.2%, Greed: 85.6%) sentiment does not affect win probability
- Worst single drawdown occurred during Greed (-$369,393), not Fear

### Q2: Do traders change behavior based on sentiment?

- Traders make 37% more trades on Fear days (105 vs 77 per day)
- Position sizes are 57% larger on Fear days ($7,182 vs $4,575 mean)
- Fee burden is nearly double on Fear days ($147 vs $77 per day)
- Counterintuitive: slight short bias during Greed (52.9%), near-neutral during Fear (49.5%)

### Q3: Trader segments

Three segments defined from actual data distributions:

**Segment A — Trading Frequency**

| Segment | Accounts | Avg trades/day | Avg PnL | Better sentiment |
|---|---|---|---|---|
| High (>138/day) | 8 | 301 | $553k | Fear (mean) |
| Mid (60–138/day) | 8 | 84 | $214k | Greed |
| Low (<60/day) | 16 | 33 | $260k | Greed |

**Segment B — Win Rate Consistency**

| Segment | Accounts | Avg PnL | % Profitable |
|---|---|---|---|
| High win rate (>95%) | 8 | $252k | 100% |
| Mid win rate (76–95%) | 16 | $489k | 100% |
| Low win rate (<76%) | 8 | $58k | 63% |

**Segment C — Activity Rate**

| Segment | Accounts | Fear median PnL | Greed median PnL |
|---|---|---|---|
| Active (>30% of days) | 5 | $94 | $392 |
| Moderate (16–30%) | 3 | $390 | $498 |
| Selective (<16%) | 24 | $87 | $0 |

---

## Part C — Strategy Recommendations

### Strategy 1 — Reduce activity and position size during Fear days

During Fear days, active and moderate traders should reduce trade frequency by at least 25% and cap position sizes at their prior Greed-period average. The data shows they are overtrading with larger sizes during Fear — paying twice the fees for 54% lower median returns. Selective traders are exempt — their low frequency already provides natural protection.

### Strategy 2 — Manage toward the 76–95% win rate band

Do not optimise purely for win rate. Mid win rate accounts (76–95%) produced the highest average PnL ($489k) and were 100% profitable. High win rate accounts (>95%) earn 48% less on average — they over-filter entries. Low win rate accounts (<76%) have 3 net losers despite win rates above 65%. If win rate exceeds 95%, widen entry criteria. If below 76%, tighten stop-losses before increasing position size.

---

## Bonus — Predictive Model

A binary classifier was trained to predict next-day trader profitability using same-day sentiment and behavior features. Three models evaluated using 5-fold stratified cross-validation.

| Model | Accuracy | Base rate | Delta |
|---|---|---|---|
| Logistic Regression | 70.5% | 69.7% | +0.8% |
| Random Forest | 63.5% | 69.7% | -6.2% |
| Gradient Boosting | 68.5% | 69.7% | -1.2% |
| RF + lagged features | 66.8% | 74.1% | -7.3% |
| Per-account RF (best) | 51.7% | 51.0% | +0.7% |

**Result:** No model meaningfully beat the majority-class baseline. Root cause: mean lag-1 PnL autocorrelation is 0.071 — essentially a random walk. Behavior features (position size, trade frequency) outrank sentiment features in importance. The raw Fear/Greed score (0–100) carries 3x more signal than the categorical label.

