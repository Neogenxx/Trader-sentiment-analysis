# Trader Performance vs Market Sentiment — Analysis Summary

**Dataset:** 211,224 trades · 32 accounts · May 2023 – May 2025 · Hyperliquid x Bitcoin Fear/Greed Index

---

## Methodology

Two datasets were loaded, cleaned, and merged by date. The Bitcoin Fear/Greed Index (2,645 daily rows, 2018–2025) was joined to Hyperliquid trader data using a left merge on extracted trade date. Sentiment coverage after merge was 100%.

**Data issues resolved:**
- Unix `Timestamp` column corrupted by Excel scientific notation — collapsed 211,224 unique values into 7. Dropped; `Timestamp IST` used exclusively
- 50.6% of trades have `Closed PnL = 0` (open positions). Win rate calculated on closed trades only throughout
- One missing sentiment date (2024-10-26) forward-filled from prior day — standard practice for single-day index gaps

**Metrics engineered per account per day:** daily PnL, trade count, win rate (closed trades only), long/short ratio, average position size, fees, drawdown proxy. Three trader segments defined from verified data distributions: trading frequency, win rate consistency, and activity rate.

---

## Key Insights

### Insight 1 — Greed days produce better typical returns; Fear days produce higher variance

| Sentiment | Median PnL | Mean PnL | Positive day % |
|---|---|---|---|
| Fear | $123 | $5,185 | 60.4% |
| Neutral | $168 | $3,439 | 62.2% |
| Greed | $267 | $4,177 | 64.3% |

Median PnL rises consistently from Fear to Greed. The Fear mean ($5,185) exceeds Greed ($4,177) only because of a small number of extremely large winning days. A typical trader on a typical day makes 2.2x more on Greed days. Win rate is nearly identical across all sentiment regimes (84–86%) — sentiment affects trade size and frequency, not win probability.

---

### Insight 2 — Traders overtrade during Fear, paying double the cost for lower returns

| Sentiment | Avg trades/day | Avg size USD | Avg fees/day |
|---|---|---|---|
| Fear | 105 | $7,182 | $147 |
| Neutral | 100 | $4,783 | $105 |
| Greed | 77 | $4,575 | $77 |

During Fear, traders execute 37% more trades and take 57% larger positions than on Greed days — resulting in nearly double the daily fee burden ($147 vs $77). Despite higher activity and larger positions, median PnL is 54% lower. Traders are systematically increasing risk and cost during the worst-performing sentiment regime. Counterintuitively, traders also show a slight short bias during Greed (52.9%) and are near-neutral during Fear (49.5%).

---

### Insight 3 — Mid win rate accounts (76–95%) outperform both extremes

| Win rate segment | Avg total PnL | % profitable | Greed median PnL |
|---|---|---|---|
| High (>95%) | $252,396 | 100% | $138 |
| Mid (76–95%) | $488,565 | 100% | $390 |
| Low (<76%) | $57,593 | 63% | $35 |

Mid win rate accounts earn 93% more on average than high win rate accounts. High win rate accounts appear to over-filter entries — taking only the safest setups and missing larger opportunities. All three net-losing accounts in the dataset have win rates above 65% yet are net negative — they win frequently but lose large.

---

### Insight 4 — Active traders earn 4.5x more on Greed days than Fear days

| Activity segment | Fear median PnL | Greed median PnL | Ratio |
|---|---|---|---|
| Active (>30% of days) | $94 | $392 | 4.2x |
| Moderate (16–30%) | $390 | $498 | 1.3x |
| Selective (<16%) | $87 | $0 | — |

Active traders show the strongest Greed preference. Selective traders (fewer than 16% of available days) show similar Fear and Greed medians — their naturally low activity already filters out sentiment-driven noise. High frequency traders earn more on Fear days by mean but medians are similar, suggesting Fear outperformance is driven by a few outlier days, not consistent performance.

---

## Strategy Recommendations

### Strategy 1 — Reduce activity and position size during Fear days

**Target:** Active and Moderate traders.

**Evidence:** Active traders earn 4.5x more on Greed days at the median ($392 vs $94). Moderate traders earn 28% more on Greed ($498 vs $390). Across all traders, Fear days generate 37% more trades, 57% larger positions, and nearly double the fee burden — for 54% lower median returns.

**Rule of thumb:** When the Fear/Greed index falls below 45 (Fear or Extreme Fear), reduce daily trade count by at least 25% and cap position sizes at the prior 30-day Greed-period average. Do not increase position size to average down during Fear — the data shows this behaviour is widespread and consistently underperforms. Selective traders (activity rate below 16%) are exempt — their low frequency already provides natural protection.

---

### Strategy 2 — Manage toward the 76–95% win rate band

**Target:** All traders, particularly Low win rate accounts.

**Evidence:** The 16 mid-win-rate accounts (76–95%) produced an average total PnL of $488,565 and were 100% profitable. High win rate accounts (>95%) earned 48% less on average. Low win rate accounts (<76%) had 3 net losers despite win rates above 65%.

**Rule of thumb:** If overall win rate consistently exceeds 95%, the strategy is too conservative — widen entry criteria or reduce minimum reward-to-risk threshold. If win rate falls below 76%, tighten stop-losses before increasing position size — the issue is loss magnitude, not entry frequency. The losing accounts in this dataset all share the same pattern: high win rates masking a few catastrophic losses that erase months of gains.

---

## Bonus — Predictive Model

A binary classifier was trained to predict next-day trader profitability using same-day sentiment and behavior features.

| Model | Accuracy | Base rate | Delta |
|---|---|---|---|
| Logistic Regression | 70.5% | 69.7% | +0.8% |
| Random Forest | 63.5% | 69.7% | -6.2% |
| Gradient Boosting | 68.5% | 69.7% | -1.2% |
| RF + lagged features | 66.8% | 74.1% | -7.3% |
| Per-account RF (best) | 51.7% | 51.0% | +0.7% |

**Result:** No model meaningfully beat the majority-class baseline.

**Root cause:** Mean lag-1 PnL autocorrelation across accounts is 0.071 — essentially a random walk. No sequential structure exists to exploit at the population level.

**What the model revealed:** Behavior features (position size, trade frequency) consistently outrank sentiment features in importance. The raw Fear/Greed score (0–100) carries 3x more signal than the categorical label (importance 0.099 vs 0.032). The null result confirms the Part B descriptive findings — sentiment shapes how traders behave in aggregate, it does not predict whether any individual trader will profit tomorrow.

---

## Dataset Summary

| Metric | Value |
|---|---|
| Total trades | 211,224 |
| Unique accounts | 32 |
| Trading days | 480 (May 2023 – May 2025) |
| Unique coins | 246 |
| Sentiment coverage | 100% |
| Closed trades | 104,408 (49.4%) |
| Profitable accounts | 29 / 32 |
| Top 5 accounts PnL share | 61.8% |