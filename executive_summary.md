# Executive Summary — raw commission-only CSV analysis

## Scope

This analysis parses the uploaded TradingView trade CSV for `BTCUSDT.P` on `15m`, baseline `risk_based_baseline_slim`, over the user-specified full backtest window `2020-01-01 → 2026-06-14`.

Cost basis: **Scenario A: commission-only 0.05% per side**. **0.05%/side commission-only excludes slippage and perpetual funding; it should not be used as a final public headline without caveats.**

This file is public-safe evidence text for the raw Scenario A analysis. It is **not** a recommended public headline.

## Data parsed

- CSV rows parsed: **1,020**
- Round-trip trades parsed: **510**
- First closed trade in CSV: **2020-07-23 14:15**
- Last closed trade in CSV: **2026-06-10 09:30**
- Initial capital inferred from cumulative PnL columns: **~1,000.00 USDT**; calculations use **1,000.00 USDT**.

## Headline context

User-verified TradingView headline for this raw commission-only run:

- Net return: **+623.91%**
- Max drawdown: **10.86%**
- Profit factor: **2.265**
- CAGR: **35.89%**
- Trades: **510**

CSV reconstruction from closed trades gives final cumulative PnL of **6,239.07 USDT** and final equity of **7,239.07 USDT**, equivalent to **+623.91%** on the inferred 1,000 USDT base. The profit factor from rounded per-trade PnL is **2.265**. The per-trade PnL sum differs from the cumulative PnL column by **0.05 USDT**, which is consistent with CSV rounding.

The drawdown chart in this pack is derived from **closed-trade cumulative PnL only** and shows **10.29%** max closed-trade drawdown. Do not use that as a replacement for the verified TradingView headline drawdown of **10.86%**, because the platform may use higher precision and/or a different drawdown convention.

## Trade quality and side attribution

Overall win rate is **43.73%** (223 winners / 287 losers). Average winner is **50.10 USDT** and average loser is **-17.19 USDT**, giving an average win/loss payoff ratio of **2.91x**. This is not a high-win-rate system; the result depends on winners being materially larger than losers.

Long-side attribution: **346 trades**, **4,216.73 USDT**, PF **2.203**, win rate **44.51%**.

Short-side attribution: **164 trades**, **2,022.39 USDT**, PF **2.417**, win rate **42.07%**.

These long/short figures are **post-hoc attribution from the full trade list**, not independent walk-forward recomputation.

## Holding-time attribution

Holding-time bucket results are also post-hoc attribution:

| Holding bucket | Trades | Net PnL | Win rate | Profit factor |
|---|---:|---:|---:|---:|
| `<6h` | 203 | -1,707.60 USDT | 31.53% | 0.446 |
| `6-24h` | 247 | 2,760.23 USDT | 44.13% | 2.542 |
| `1-3d` | 60 | 5,186.49 USDT | 83.33% | 82.115 |
| `>3d` | 0 | 0.00 USDT | n/a | n/a |

The sub-6h bucket is net negative, while the 1-3 day bucket carries most of the net result. That is a useful risk note: execution quality and missed runners matter more than the headline trade count alone suggests.

## Fat-tail dependency

Top-trade dependency, using rounded per-trade Net PnL from the CSV:

| Removed best trades | PnL of removed trades | Share of total net PnL | Net return without them | PF without them |
|---:|---:|---:|---:|---:|
| 1 | 651.12 USDT | 10.44% | 558.80% | 2.133 |
| 5 | 2,860.66 USDT | 45.85% | 337.85% | 1.685 |
| 10 | 4,238.55 USDT | 67.94% | 200.06% | 1.405 |

This is a fat-tailed profile. The strategy can remain profitable without the single best trade, but the best 5-10 trades explain a large share of the raw net PnL. Public wording should say this plainly.

Longest observed winning streak: **10 trades**. Longest observed losing streak: **18 trades**.

## Monte Carlo reshuffle check

Monte Carlo method: **10,000 random reshuffles without replacement** of the observed rounded per-trade Net PnL sequence, starting from 1,000.00 USDT. This tests path dependency under the observed trade distribution; it does **not** model new regimes, missed fills, slippage, funding, or parameter decay.

- Median reshuffled max drawdown: **17.78%**
- 95th percentile reshuffled max drawdown: **38.45%**
- 99th percentile reshuffled max drawdown: **53.27%**
- Median reshuffled max losing streak: **10 trades**
- 95th percentile reshuffled max losing streak: **14 trades**
- 99th percentile reshuffled max losing streak: **17 trades**

## Buyer-safe wording

A conservative public-safe sentence for this raw run:

> From 2020-01-01 to 2026-06-14, the frozen BTCUSDT.P 15m baseline produced a TradingView commission-only backtest of +623.91%, 10.86% max drawdown, PF 2.265, and 510 trades. This excludes slippage and perpetual funding, is not a live-return expectation, and should be read as raw backtest evidence rather than a final public headline.

Monthly and yearly tables in this pack are **period attribution from the full exported trade list**, not true walk-forward recomputation.
