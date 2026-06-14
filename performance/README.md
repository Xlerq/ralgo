# ralgo - Performance Pack

Public, reproducible performance evidence for the **ralgo** BTC-futures strategy.

> **Status: HEADLINE + COSTS VERIFIED.** The headline BTC Deep Backtest and the cost
> sensitivity (A/B/C) are reproduced and poll-to-stable. Funding is estimated analytically.
> Scenario A attribution tables/charts are generated from the exported trade list. Walk-forward
> and exit-reason attribution are still pending. See [`public_metrics.json`](public_metrics.json).

## Candidate under test

| Field | Value |
|---|---|
| Candidate version | **`risk_based_baseline_slim`** (frozen — no tuning, no parameter changes) |
| Sides | long + short (both-sides) |
| Instrument | `BINANCE:BTCUSDT.P` (BTC USDⓂ perpetual futures) |
| Base timeframe | 15m |
| Engine | TradingView **Deep Backtesting**, full range |
| Date range | 2020-01-01 → 2026-06-14 (effective start ~2020-07 after trend-filter warmup) |
| Effective initial capital | 1000 |

The strategy is treated as **frozen**. This pack measures it; it does **not** optimize,
tune, or search for better settings.

## Verified Result

**The current verified result is BTC, both-sides, in-sample.** It is a backtest, not a live
return, and cross-asset transfer is **not** verified for this pack (see `limitations.md`).

| Cost basis | Net | Max DD | Profit factor | CAGR | Source |
|---|---|---|---|---|---|
| Commission only (0.05%/side) — **measured** | +623.91% | 10.86% | 2.265 | 35.89% | A |
| Fee + slippage (0.07%/side) — **measured, recommended headline** | +518.40% | 11.39% | 2.091 | 32.61% | B |
| Stress (0.10%/side) — **measured** | +388.14% | 12.33% | 1.865 | 27.84% | C |
| + perpetual funding — **ESTIMATED** | ~+375% to +450% | ~11–12% | ~1.8–2.0 | ~27–30% | est. |

510 trades, all scenarios. Realistic costs remove a **~40% central bite** (range ~22–52%) from
the commission-only figure. Full detail: [`tables/cost_sensitivity.csv`](tables/cost_sensitivity.csv).

`public_metrics.json` is the single machine-readable public metrics contract. The older root-level
commission-only JSON has been removed so consumers do not have two conflicting truth sources.

## Public Charts

- [`charts/equity_curve.png`](charts/equity_curve.png)
- [`charts/drawdown_curve.png`](charts/drawdown_curve.png)
- [`charts/monthly_returns_heatmap.png`](charts/monthly_returns_heatmap.png)
- [`charts/trade_distribution.png`](charts/trade_distribution.png)

## Layout

```text
performance/
├── README.md            ← you are here
├── methodology.md       ← how the pack is produced (config, settings, cost model, repro rule)
├── limitations.md       ← honest red-team: overfitting, funding-estimate risk, paper-vs-live
├── public_metrics.json  ← machine-readable headline + cost-adjusted metrics
├── raw_runs/            ← raw per-run exports + stability poll logs (baseline, cost sensitivity)
├── tables/              ← cost sensitivity + Scenario A attribution CSV tables
├── simulations/         ← Monte Carlo reshuffle output
├── charts/              ← equity curve, drawdown, heatmap, distribution charts
└── audit_logs/         ← reproducibility logs: poll-to-stable evidence, INVALID-run records
```

## Reproducibility rule (non-negotiable)

After **every** symbol switch, source update, compile, or Deep Backtest run, the headline
metrics and trade count must be **polled until stable across consecutive reads** before being
recorded. If stable results cannot be obtained, the run is marked **INVALID** and discarded —
never recorded as partial/stale output. See [`methodology.md`](methodology.md).

## Public-repo safety

This is a **public** repository. It contains methodology, assumptions, metrics, CSV tables,
charts, and JSON summaries only. It contains **no** private Pine Script source, signal-logic
thresholds, secrets, webhook URLs, account IDs, or API keys.

## Risk notice

Trading leveraged crypto futures involves high risk. Backtests do not guarantee future
performance. Nothing here is financial advice. For informational/research purposes only.
