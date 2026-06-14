# Limitations — raw commission-only CSV analysis

## 1. Cost model is incomplete

This analysis uses **Scenario A: commission-only 0.05% per side** because that is the cost basis of the uploaded trade CSV. It **excludes slippage and perpetual funding**. For BTCUSDT perpetual futures, funding can materially change results, especially for positions held across multiple funding intervals. Slippage can also be concentrated in volatile breakout periods, exactly where the largest winners and losers tend to occur.

Therefore this Scenario A output should not be used as the final public headline without prominent caveats. It is raw commission-only evidence.

## 2. Trade-list attribution is not walk-forward recomputation

Monthly returns, yearly returns, long/short attribution, holding-time buckets, best/worst trades, and top-trade dependency are all derived from the **final full exported trade list**. They are useful descriptive attribution, but they are **not** separate walk-forward recomputations, not out-of-sample tests, and not evidence that parameters were stable when viewed only up to each period.

Use the phrase: **"period attribution from the full trade list, not true walk-forward recomputation."**

## 3. Drawdown curve is closed-trade equity only

The CSV provides cumulative PnL at closed trades. It does not provide full intrabar equity, floating position equity, liquidation-path information, or account-level mark-to-market exposure. The generated drawdown curve is therefore based on closed-trade cumulative PnL only.

CSV-derived closed-trade max drawdown in this pack is **10.29%**. The user-verified TradingView headline max drawdown is **10.86%**. Prefer the verified TradingView value when quoting the platform headline; use the CSV curve only as a closed-trade reconstruction.

## 4. Per-trade PnL values are rounded

The sum of rounded `Net PnL USDT` values is **6,239.12 USDT**, while the final `Cumulative PnL USDT` column is **6,239.07 USDT**. The difference is **0.05 USDT**, consistent with CSV precision/rounding. Period return tables use cumulative PnL differences for equity returns; distribution, dependency, and Monte Carlo analysis use the rounded per-trade PnL values because those are the trade-level values available in the CSV.

## 5. Monte Carlo reshuffling has narrow meaning

The Monte Carlo file contains **10,000 reshuffles** of the observed trade PnL sequence. This preserves the observed trade outcomes but changes their order. It is a path-dependency test, not a market simulator.

It does not model:

- missed signals,
- live fill degradation,
- slippage,
- funding,
- exchange outages,
- latency,
- order-book depth,
- regime changes,
- parameter decay,
- alternative histories where the strategy generated different trades.

## 6. Holding-time buckets are descriptive, not tradable rules

The holding-time result is strongly skewed: the `<6h` bucket is net negative and the `1-3d` bucket carries most of the net PnL. This does **not** prove that filtering by holding time would improve live performance, because holding time is known only after the trade closes. Treat it as descriptive risk attribution, not as an executable rule.

## 7. Missing fields that were not estimated

The CSV does not contain measured fields for:

- slippage,
- perpetual funding paid/received,
- maker/taker fill mix,
- partial fills,
- order-book liquidity,
- latency,
- intrabar mark-to-market equity,
- true walk-forward period-by-period recomputation,
- live or paper-forward execution results.

No values for those items were invented in this output.

## 8. Public presentation constraint

The raw +623.91% figure is a commission-only backtest number. Public copy should avoid implying expected live performance. A skeptical-buyer-safe presentation should place the cost caveat near the metric, not in a footnote hidden below the chart.
