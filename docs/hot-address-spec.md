# Hot Address Spec (Selection Algorithm)

## TL;DR (Too Long; Didn't Read)

- Goal: for a given user capital tier, select “hot” leader addresses that are **valuable and actually copyable**
- Pipeline: candidate pool → baseline filters → compute 4 factors → tier-specific weighting → rank → Top N
- Factors (0–1): **return**, **risk**, **executability**, **position-size fit**
- Key focus: reduce “can’t open” outcomes for smaller accounts (DEX minimum order size / mismatched position sizing), not just high PnL/ROI

---

## 1) Goal and approach

For users with capital between **100 and 100,000 U (USDT equivalent)**, output a list of “Hot Addresses” that better match the user’s capital tier for copy trading.

High-level approach:
- Fetch a candidate address pool (with trading statistics)
- Apply baseline filters to remove low-quality / unrealistic addresses
- Compute four core factors per address (0–1)
- For the user’s capital tier, apply tier-specific filters and a weighted scoring formula
- Sort by overall score and return Top N

---

## 2) Inputs

- `user_capital`: user capital (U / USDT equivalent)
- `lookback_days`: time window (e.g. 7 / 30 / 90)
- `chain`: chain filter (e.g. `all` / `eth` / `bsc` / ...)
- `risk_level`: optional (default `balanced`; can be used to override tier defaults)
- `limit`: number of addresses to return

---

## 3) Capital tiers

Supported capital range:
- Min: **100 U**
- Max: **100,000 U**

Tier mapping (configurable):
- **Tier 1**: 100 – 1,000 U
- **Tier 2**: 1,000 – 10,000 U
- **Tier 3**: 10,000 – 100,000 U

Implementation notes:
- Keep raw `user_capital` for precise calculations
- Derive `capital_tier` to select tier thresholds and weights

---

## 4) Baseline filters (apply to all tiers)

Only addresses passing all baseline filters enter scoring.

### 4.1 Activity

Requirements:
- `total_trades >= 30`
- `active_days >= lookback_days * 0.6`

Rationale: ensure sufficient samples and consistent activity.

### 4.2 Profitability + risk floor

Requirements:
- `roi_total >= 0.10` (>= 10% total ROI)
- `max_drawdown >= -0.50` (drawdown not worse than -50%)
- `win_rate >= 0.45` (>= 45% win rate)

Rationale: remove clearly losing or extremely unstable addresses.

### 4.3 Trading cadence

Requirements:
- `avg_trades_per_day <= 30`
- `avg_hold_hours >= 0.25` (>= 15 minutes average holding time)

Rationale: avoid ultra-HFT / “too fast to follow” styles for typical users.

---

## 5) Core factors (0–1)

After baseline filtering, compute four factors per address:
- `position_size_factor` (position-size fit)
- `risk_factor`
- `execution_factor` (executability)
- `return_factor`

These are combined into an overall score using tier-specific weights.

### 5.1 Position-size fit: `position_size_factor`

Measures whether the address’s typical single-trade position size matches the user’s capital.

Inputs:
- `user_capital`
- `median_position_size`: median single-trade position size (USDT equivalent) over the lookback window
- `single_trade_ratio`: max acceptable per-trade ratio for users (global parameter; default 0.2 / 20%)

Plain-text formula:
- `max_affordable_position = user_capital * single_trade_ratio`
- If `median_position_size <= max_affordable_position`: `position_size_factor = 1`
- Else:
  - `ratio = max_affordable_position / median_position_size`
  - `position_size_factor = clamp(ratio, 0, 1)`

Why median: more robust against outliers than mean.

### 5.2 Risk: `risk_factor`

Combines drawdown and win-rate.

Inputs:
- `max_drawdown` (negative values, e.g. -0.25 means -25%)
- `win_rate` (0–1)

Steps:
1) Drawdown score:
- `md_clipped = clamp(max_drawdown, -0.8, 0)`
- `risk_dd_score = 1 - (abs(md_clipped) / 0.8)`

2) Win-rate score:
- `wr_clipped = clamp(win_rate, 0.4, 0.8)`
- `risk_wr_score = (wr_clipped - 0.4) / 0.4`

3) Combine:
- `risk_factor = 0.5 * risk_dd_score + 0.5 * risk_wr_score`

### 5.3 Executability: `execution_factor`

Measures whether typical users can keep up with the trading rhythm.

Inputs:
- `avg_trades_per_day`
- `avg_hold_hours`

Steps:
1) Frequency score:
- `t_clipped = clamp(avg_trades_per_day, 1, 30)`
- Use 5 trades/day as a tunable “sweet spot”:
  - `freq_score = 1 - abs(t_clipped - 5) / 25`
  - `freq_score = clamp(freq_score, 0, 1)`

2) Holding-time score (piecewise recommendation):
- `h_clipped = clamp(avg_hold_hours, 0.25, 72)`
- If `0.25 <= h_clipped <= 2`: linearly increase from 0.5 to 1
- If `2 < h_clipped <= 24`: `hold_score = 1`
- If `24 < h_clipped <= 72`: linearly decrease from 1 to 0.5

3) Combine:
- `execution_factor = 0.5 * freq_score + 0.5 * hold_score`

### 5.4 Return: `return_factor`

Captures ROI but clips extreme values to reduce “blow-up later” leaderboard gaming.

Inputs:
- `roi_total` (e.g. 0.6 means +60%)

Plain-text formula:
- `roi_clipped = clamp(roi_total, 0, 2)` (0–200%)
- `return_factor = roi_clipped / 2`

---

## 6) Tier-specific filters and weights

### 6.1 Tier 1 (100 – 1,000 U)

Profile: smaller capital, lower risk tolerance, stronger focus on stability and followability.

Additional filters:
- `avg_trades_per_day <= 15`
- `max_drawdown >= -0.40`
- Optional: `median_position_size <= user_capital * 0.5`

Weights:
- `return_factor`: 20%
- `risk_factor`: 40%
- `execution_factor`: 25%
- `position_size_factor`: 15%

Overall score:

```text
score_overall = 100 * (
  0.20 * return_factor +
  0.40 * risk_factor +
  0.25 * execution_factor +
  0.15 * position_size_factor
)
```

### 6.2 Tier 2 (1,000 – 10,000 U)

Profile: core users balancing return and risk.

Additional filters: baseline only.

Weights:
- `return_factor`: 30%
- `risk_factor`: 35%
- `execution_factor`: 20%
- `position_size_factor`: 15%

```text
score_overall = 100 * (
  0.30 * return_factor +
  0.35 * risk_factor +
  0.20 * execution_factor +
  0.15 * position_size_factor
)
```

### 6.3 Tier 3 (10,000 – 100,000 U)

Profile: larger capital, willing to accept more volatility for higher return.

Additional filters (slightly looser on drawdown):
- `max_drawdown >= -0.60`

Weights:
- `return_factor`: 40%
- `risk_factor`: 30%
- `execution_factor`: 15%
- `position_size_factor`: 15%

```text
score_overall = 100 * (
  0.40 * return_factor +
  0.30 * risk_factor +
  0.15 * execution_factor +
  0.15 * position_size_factor
)
```

---

## 7) Capital suitability score (normalized output)

Independent of tier, output a capital-suitability score:

```text
score_suitability_for_capital = 100 * position_size_factor
```

Range: 0–100.

Frontend copy examples:
- “Capital fit: 82/100, suggested minimum capital >= 500 U”
- “Typical single trade: 80 U; your capital: 400 U; good match for copying”

---

## 8) Output fields

- `address`, `chain`
- `stats`:
  - `roi_total`, `pnl_total`, `max_drawdown`, `win_rate`
  - `total_trades`, `active_days`
  - `avg_trades_per_day`, `avg_hold_hours`
  - `median_position_size`, `max_position_size`
- `scores`:
  - `return_factor`, `risk_factor`, `execution_factor`, `position_size_factor`
  - `score_overall`
  - `score_suitability_for_capital`

---


