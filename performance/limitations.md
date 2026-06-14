# Limitations & Red-Team Notes

This pack is written by a hostile reviewer of its own results. Read this **before** the
headline numbers. Every item here is a reason the live result may be worse than the backtest.

## 1. Overfitting / cross-asset failure (the big one)

- The strategy's parameters are dimensionless (ATR- and ratio-relative), so on paper they
  *should* transfer to other assets. **They don't.**
- BTC in-sample profit factor is ~2.1–2.3. On the most-comparable assets the edge collapses:
  - ETH ≈ PF 1.22–1.34,
  - SOL ≈ PF 1.24.
- A ~1.65× BTC-vs-ETH PF gap on dimensionless params is a **strong overfitting signal**. The
  impressive BTC number should be read as **BTC-specific**, not a general edge.
- **Implication:** this is treated as a **BTC-only** strategy. Cross-asset deployment is *not*
  supported by the evidence, and real-capital risk should be sized against the weaker
  cross-asset PF, not the headline BTC PF.

## 2. Funding is ESTIMATED, not measured

- TradingView cannot model perpetual funding. The funding cost layer is **analytical**.
- It is anchored on a cost-sensitivity **measured on this candidate** (≈ **−24 net pts per
  0.01% round-trip**, from the A/B/C deep runs) but the funding *rate* and the *notional-weighted
  hold time* are assumptions, and both plausibly push costs **higher** (a long-in-uptrend book
  pays above-average funding; the biggest runners are held longest).
- Commission + slippage alone (measured) already removes **16.9%** of the commission-only net.
  With estimated funding, costs take a **~22–52% bite (central ~40%)** out of the in-sample paper
  return. Live funding could exceed the conservative end in a high-funding regime.
- Only the commission/slippage scenarios (A/B/C) are **measured**; the **funding** layer is
  **ESTIMATED**. See `tables/cost_sensitivity.csv` and `audit_logs/cost_sensitivity.md`.

## 3. Paper / backtest vs live divergence

- All numbers are **backtest** numbers. No live or forward-tested fills are included here.
- Backtest fills assume the modeled commission+slippage; real taker fills during volatility,
  liquidations, and thin books can be worse.
- Order-type assumptions, partial fills, and exchange outages are not modeled.

## 4. Fat-tailed return profile = fragile to a few trades

- The edge is concentrated: a small number of multi-day "runner" trades carry the majority of
  net profit, while sub-6h churn is net-negative.
- This means the headline return is **sensitive to a handful of trades**. Miss or mistime the
  big runners (slippage on entry, an early stop, a missed signal in live) and the realized
  result degrades faster than the trade count suggests.
- Median trade is around break-even/slightly negative; the mean is carried by the tail.

## 5. Sample size & regime dependence

- Total round-trips over the full history are in the **few-hundreds**, not thousands — adequate
  but not large, especially once split by year/side/duration.
- Performance is regime-dependent. The daily trend filter keeps the book mostly flat through
  the 2022 bear; the strong years (e.g. 2025) dominate aggregate return. A prolonged
  chop/sideways regime is the untested adverse case.

## 6. Measurement / tooling caveats

- TradingView Deep Backtest can redisplay **stale partial snapshots** mid-recompute. One such
  phantom (a 178-trade ETH snapshot) was already caught and discarded in this project. The
  poll-to-stable rule (see `methodology.md` §6) exists specifically to defend against this; any
  number that wasn't polled to stability is suspect.
- Deep results are read from the Strategy Tester DOM, not the internal data APIs (which return
  zero/standard-tester values for deep runs).

## 7. Warmup / start-date effect

- The configured range starts 2020-01-01 but the daily trend filter is invalid until ~mid-2020,
  so the **effective** start is ~2020-07. Returns and trade counts implicitly exclude the first
  ~6 months. Any annualized/CAGR figure should be computed off the effective start.

## 8. Candidate-vs-published-data consistency (open item)

- The existing public `../data/` CSVs were generated from an earlier trade export
  (524 exit trades, PF ~2.07). The frozen `risk_based_baseline_slim` deep reading is
  PF 2.265 / 510 trades (commission-only) — or PF 2.091 at the recommended fee+slippage basis.
  These are close but **not identical** and use different units (USDT vs %) and cost bases.
- Until the existing CSVs are regenerated from this candidate (or explicitly reconciled), treat
  the two sources as **provisional and not yet cross-checked**.

---

**Bottom line:** the BTC backtest shows a real, temporally-robust long edge, but (a) it is
BTC-specific, (b) realistic costs remove ~40% of the paper return (measured fee+slippage −16.9%,
plus estimated funding), and (c) it is fat-tailed and therefore fragile to execution. Set live
expectations well below the headline backtest figure.
