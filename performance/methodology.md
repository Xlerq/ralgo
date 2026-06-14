# Methodology

How the ralgo performance pack is produced. The goal is **honest, reproducible measurement**
of a **frozen** strategy — not optimization.

---

## 1. Candidate version

- **`risk_based_baseline_slim`** (Engulfing_TURBO, RiskBased — slim harness). This is the frozen
  candidate for the public performance pack. It is the **both-sides** (long + short) baseline;
  the slim harness reproduces the original RiskBased baseline to the cent.
- **Frozen contract:** parameters are not changed, not tuned, and not searched. The only value
  changed in this pack is the **cost assumption** (`commission_value`) when measuring cost
  scenarios — strategy logic and all signal/risk parameters stay fixed. If a result looks
  improvable, that is noted in `limitations.md` — it is **not** acted on here.

## 2. Instrument & timeframe

| Field | Value |
|---|---|
| Symbol | `BINANCE:BTCUSDT.P` |
| Market | BTC USDⓂ perpetual futures (Binance) |
| Base timeframe | 15-minute |
| Higher-timeframe context | a daily trend filter is used (drives the warmup caveat below) |

Cross-asset behavior (ETH, SOL) is documented in `limitations.md`, not published as headline
performance. The candidate is a **BTC** instrument; ETH/SOL are robustness probes only.

## 3. TradingView Deep Backtest settings

| Setting | Value |
|---|---|
| Mode | **Deep Backtesting** (full history, not the standard ~limited-bar tester) |
| Initial capital | **1000** (effective). The script declares 100; the Strategy Tester *Properties* override it to 1000. % metrics are scale-invariant (risk/equity-relative sizing), so this affects only absolute USDT. |
| Position sizing | risk-based (a fixed % of equity risked per trade), with a leverage cap |
| Pyramiding | **3** (strategy value) |
| Commission | **0.05% per side** (Binance USDⓂ futures taker) — modeled in every run |
| Slippage | modeled as an **additional 0.02% per side** (see §5), i.e. 0.07%/side all-in |
| Recalculate | after order is filled / on every tick as per default strategy behavior |

> Exact entry/exit signal thresholds, the risk-% value, the leverage cap, and the
> core/runner split are **private** and deliberately omitted from this public document.

## 4. Date range

- **Configured range:** 2020-01-01 → 2026-06-14.
- **Effective start:** ~2020-07. The daily trend filter needs history to warm up, so the
  earliest months produce no/garbage signals. The first reliable trades begin ~2020-07
  (consistent with the existing public trade export starting 2020-07-23).
- Walk-forward windows (when run) split this range into non-overlapping temporal blocks
  covering the 2020–21 bull, 2022–23 bear+recovery, and 2024–26 bull regimes.

## 5. Cost assumptions (currently known)

Costs are layered so each layer is independently auditable:

1. **Commission — 0.05% per side.** Binance USDⓂ futures taker fee. This has been modeled in
   every deep run from the start; the headline backtest is *already* commission-inclusive.

2. **Slippage — +0.02% per side**, folded in as additional commission → **0.07% per side**,
   i.e. **0.14% of notional per round-trip** (Scenario B). A **stress** scenario adds +0.05%/side
   → 0.10%/side, 0.20%/round-trip (Scenario C).
   - Modeled as a **percentage**, not a fixed tick. A fixed-tick slippage cannot hold a
     constant percentage across BTC's $4k → $110k price range over 2020–2026, so a %-based
     model is the defensible choice. TradingView supports percent commission, so slippage is
     added as extra percent commission per side.
   - **Measured on this candidate (A/B/C deep runs, poll-to-stable):** net return fell
     **≈ −24 percentage points per 0.01% of round-trip cost** (compounding-inclusive; range
     −22 to −26, non-linear). **Slippage alone (A→B) removed 16.9% of the commission-only net.**
     This both-sides candidate (510 trades) is more cost-sensitive than the earlier long-only
     probe (−14) because of higher turnover.

3. **Perpetual funding — ESTIMATED, not measured.** TradingView has no perp-funding model, so
   funding is computed analytically and reported as a separate layer:
   - Binance BTCUSDT perp funding historical mean ≈ **+0.01%/8h** (≈ +11%/yr). Tested at
     **0.01 / 0.02 / 0.03%/8h**.
   - Treated as a **cost on held notional regardless of side** (conservative): a both-sides trend
     book tends to pay on both legs (longs in positive-funding uptrends, shorts in
     negative-funding downtrends).
   - Per-position funding = `f_8h × (hold_hours / 8)`, converted to a net-return impact using the
     **measured −24 pts / 0.01%-round-trip** sensitivity so compounding is handled like commission.
   - Hold time uses **count-weighted ~11.25h** (avg 45 bars) and **notional-weighted ~24h** (the
     runner leg carries ~95% of notional and is held 1–3 days, so funding dollars are dominated
     by the runners — the conservative, more realistic anchor).
   - **Result:** all-in (Scenario B + estimated funding) ≈ **+375% (central) to +450%**, CAGR
     ~27–30%/yr. Central anchor = B + 0.02%/8h, notional-weighted = **+374% / CAGR 27.3%**.

> **Measured vs estimated:** Scenarios A/B/C (net, MaxDD, PF, win rate, trades) are **measured**
> TradingView deep results, each polled to stability. The **funding** layer is **estimated**
> analytically and labeled as such. Full data: `tables/cost_sensitivity.csv`,
> `raw_runs/cost_sensitivity.json`, `audit_logs/cost_sensitivity.md`.

## 6. Reproducibility rule (poll-to-stable)

TradingView's Deep Backtest widget can **redisplay a cached, partially-recomputed snapshot**
mid-recompute — with a legitimate-looking full-range/DEEP label. This has already produced one
confirmed phantom number in this project (a stale 178-trade ETH snapshot displayed during a
symbol switch, later proven false against three clean recomputes).

Therefore, for **every** symbol switch, source update, compile, or Deep Backtest run:

1. Trigger the recompute.
2. Read headline metrics **and trade count**.
3. Read again. Repeat until **two consecutive reads agree** (count + headline metrics stable).
4. Only then record the run, and save the poll trace to `audit_logs/`.
5. If stability cannot be reached, mark the run **INVALID**, discard it, and log the failure —
   never record partial/stale output.

`data_get_strategy_results` / `data_get_trades` (the MCP/internal APIs) **cannot** read deep
results (they return standard-tester or zero values); deep metrics are read from the Strategy
Tester DOM. This is documented so anyone reproducing the pack uses the same read path.

## 7. Public-repo safety rule

This document and everything in `performance/` is **public**. It must never contain:

- private Pine Script source or pseudocode of the signal logic,
- exact entry/exit thresholds, the risk-% value, leverage cap, or core/runner split,
- secrets, webhook URLs, account IDs, API keys, broker credentials.

Permitted: methodology, assumptions, aggregate metrics, CSV tables, charts, JSON summaries, and
screenshots that do not reveal private source. Raw strategy source and working notes stay in the
**private** workspace only.

## 8. Outputs produced by the pack (when run)

- `raw_runs/` — one record per deep run: headline metrics + the poll-to-stable trace.
- `tables/` — yearly, by-side, by-duration, by-exit-reason, and cost-adjusted CSV tables.
- `charts/` — equity curve, drawdown, trade-distribution charts.
- `audit_logs/` — stability evidence and any INVALID-run records.
- `public_metrics.json` — machine-readable headline summary (status flips to `VERIFIED`).
